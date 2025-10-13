---
tags:
  - docker
---
Пример использования Supervisor для запуска контейнера с несколькими службами.
# **Сценарий**

Представим, что у нас есть веб-приложение на Python (например, Flask или Django), которое для своей работы требует:
1. **Gunicorn** — для запуска самого веб-приложения.
2. **Nginx** — в качестве обратного прокси-сервера (reverse proxy) для Gunicorn.
3. **Celery Worker** — для выполнения фоновых задач.

Все эти три процесса должны работать одновременно внутри одного Docker-контейнера. Supervisor идеально подходит для управления этими процессами.

# **Шаг 1: Структура проекта**

Создадим следующую структуру папок и файлов:

```
my-multi-service-app/
├── app/
│   ├── __init__.py
│   └── main.py         # Наше простое Flask-приложение
├── supervisor/
│   └── supervisord.conf  # Конфигурационный файл Supervisor
├── nginx/
│   └── nginx.conf        # Конфигурационный файл Nginx
└── Dockerfile
```

# **Шаг 2: Файлы Конфигурации И приложения**

## **1. Приложение (`app/main.py`)**

Это простое Flask-приложение для демонстрации.
```Python
from flask import Flask
import time

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Привет от Flask, Gunicorn и Nginx!'

@app.route('/long-task')
def long_task():
    # Здесь могла бы быть логика запуска фоновой задачи через Celery
    return "Задача запущена в фоне (симуляция)!"

if __name__ == '__main__':
    app.run(host='0.0.0.0')
```

## **2. Конфигурация Nginx (`nginx/nginx.conf`)**

Nginx будет слушать порт 80 и перенаправлять запросы на Gunicorn, который будет работать на порту 8000.
```Nginx
events {
    worker_connections 1024;
}

http {
    server {
        listen 80;
        server_name localhost;

        location / {
            proxy_pass http://127.0.0.1:8000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

## **3. Конфигурация Supervisor (`supervisor/supervisord.conf`)**
Это ключевой файл. Здесь мы определяем, какие программы (службы) должен запускать и контролировать Supervisor.
```TOML
[supervisord]
nodaemon=true       ; Запускает Supervisor на переднем плане, что необходимо для Docker

[program:gunicorn]
command=/usr/local/bin/gunicorn --workers 3 --bind 0.0.0.0:8000 app.main:app
directory=/app
autostart=true
autorestart=true
stderr_logfile=/var/log/gunicorn.err.log
stdout_logfile=/var/log/gunicorn.out.log

[program:nginx]
command=/usr/sbin/nginx -g "daemon off;"
autostart=true
autorestart=true
stderr_logfile=/var/log/nginx.err.log
stdout_logfile=/var/log/nginx.out.log

[program:celery]
# В реальном проекте команда была бы сложнее, например: celery -A your_project.celery worker -l info
command=sh -c "echo 'Celery worker running...' && sleep infinity" ; Простая симуляция для примера
directory=/app
autostart=true
autorestart=true
stderr_logfile=/var/log/celery.err.log
stdout_logfile=/var/log/celery.out.log
```

**Разбор конфигурации `supervisord.conf`:**

- `[supervisord]`: Основная секция. `nodaemon=true` — очень важная настройка для Docker. Она заставляет Supervisor работать не как фоновый процесс (демон), а как основной процесс контейнера. Если Supervisor уйдет в фон, Docker посчитает, что контейнер завершил свою работу, и остановит его.
    
- `[program:НАЗВАНИЕ_СЛУЖБЫ]`: Определяет отдельную службу.
    
    - `command`: Команда для запуска службы. Обратите внимание на `nginx -g "daemon off;"` — это тоже необходимо, чтобы Nginx не уходил в фон.
    - `directory`: Рабочая директория для команды.
    - `autostart`: Запускать ли службу при старте Supervisor.
    - `autorestart`: Перезапускать ли службу, если она "упала".
    - `stdout_logfile`, `stderr_logfile`: Пути к логам стандартного вывода и вывода ошибок. Это очень удобно для отладки.

## Если Нужно Будет Накатить Миграции В БД Перед Запуском Приложений
Просто запустить сервисы недостаточно — нужно убедиться, что состояние базы данных (ее схема) соответствует тому, чего ожидает код.
Запуск миграций — это **одноразовая задача**, которая должна выполниться **до** запуска основных, долгоживущих процессов (таких как Gunicorn). Supervisor не очень хорошо подходит для таких одноразовых задач, так как он предназначен для управления _постоянно работающими_ процессами. Если вы добавите миграцию как `[program]`, Supervisor будет пытаться перезапустить ее после успешного завершения.

Поэтому правильным решением будет использовать **скрипт-точку входа (entrypoint script)**.
### **Идея**
1. Мы создадим небольшой shell-скрипт (`entrypoint.sh`).
2. Этот скрипт будет запускаться первым при старте контейнера.
3. Внутри скрипта мы:
    a. Сначала дождемся, пока база данных станет доступна. Это критически важно, так как контейнер с приложением может запуститься быстрее, чем контейнер с БД.
    b. Затем выполним команду для накатывания миграций (например, flask db upgrade или python manage.py migrate).
    c. И только после этого передадим управление основному процессу, который мы хотели запустить изначально, — Supervisor.
4. Мы обновим наш `Dockerfile` и `docker-compose.yml`, чтобы все это заработало вместе.
---

### **Шаг 1: `docker-compose.yml`**

Давайте добавим в наш `docker-compose.yml` сервис базы данных (например, PostgreSQL) и настроим переменные окружения.
```YAML
version: '3.9'

services:
  app:
    container_name: my-multi-service-app
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:80"
    volumes:
      - ./app:/app
    environment:
      # Переменные для подключения к БД. Приложение должно их использовать.
      - DATABASE_URL=postgresql://user:password@db:5432/mydatabase
    depends_on:
      # Говорим, что наш 'app' сервис зависит от 'db'.
      # Compose запустит 'db' перед 'app'.
      # Внимание: depends_on ждет только запуска контейнера, а не готовности БД!
      db:
        condition: service_started
    restart: unless-stopped

  db:
    # Используем официальный образ Postgres
    image: postgres:14-alpine
    container_name: my-app-db
    environment:
      # Эти переменные используются образом Postgres для инициализации БД
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=mydatabase
    volumes:
      # Сохраняем данные БД между перезапусками контейнера
      - postgres_data:/var/lib/postgresql/data
    ports:
      # Можно открыть порт для подключения к БД с хоста (для отладки)
      - "5433:5432"

volumes:
  postgres_data:
    # Создаем именованный volume для данных БД
```

### **Шаг 2: Создание Скрипта `entrypoint.sh`**

Создайте этот файл в корневой директории вашего проекта.
```Bash
#!/bin/sh

# Выходим из скрипта, если любая команда завершится с ошибкой
set -e

# Разбираем URL базы данных на хост и порт
# Пример: postgresql://user:password@db:5432/mydatabase
# Нам нужны 'db' и '5432'
DB_HOST=$(echo $DATABASE_URL | awk -F'[@:/]' '{print $7}')
DB_PORT=$(echo $DATABASE_URL | awk -F'[:/]' '{print $6}')

echo "Ожидание запуска PostgreSQL на хосте ${DB_HOST} и порту ${DB_PORT}..."

# Цикл ожидания доступности порта БД. Используем netcat (nc).
# pg_isready - более надежный способ для Postgres. Установите postgresql-client для этого.
while ! nc -z $DB_HOST $DB_PORT; do
  sleep 0.1
done

echo "PostgreSQL запущен!"

# Теперь, когда БД доступна, накатываем миграции.
# Замените эту команду на свою.
echo "Применение миграций базы данных..."
flask db upgrade # Например, для Flask-Migrate
# или: python manage.py migrate # Например, для Django

echo "Миграции применены."

# exec "$@" - это ключевая команда.
# Она заменяет текущий процесс (скрипт) на команду,
# переданную в CMD Dockerfile (в нашем случае, supervisord).
# Это нужно, чтобы supervisord стал главным процессом (PID 1) в контейнере.
exec "$@"
```

### **Шаг 3: Обновление `Dockerfile`**

Нам нужно добавить в `Dockerfile` команды для копирования нашего нового скрипта и указать, что он должен быть точкой входа.
```Dockerfile
# 1. Выбираем базовый образ
FROM python:3.9-slim

# 2. Устанавливаем переменные окружения
ENV PYTHONUNBUFFERED 1

# 3. Устанавливаем необходимые пакеты
# Добавляем netcat-openbsd для проверки доступности порта и postgresql-client для pg_isready
RUN apt-get update && apt-get install -y \
    supervisor \
    nginx \
    curl \
    netcat-openbsd \
    && rm -rf /var/lib/apt/lists/*

# 4. Устанавливаем Python-зависимости (добавьте нужные для миграций)
# Например, Flask-Migrate, psycopg2-binary
RUN pip install flask gunicorn flask-migrate psycopg2-binary

# 5. Копируем код приложения
COPY ./app /app

# 6. Копируем конфигурационные файлы
COPY ./supervisor/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY ./nginx/nginx.conf /etc/nginx/nginx.conf

# 7. Создаем директории для логов
RUN mkdir -p /var/log/gunicorn /var/log/nginx /var/log/celery

# 8. Копируем и делаем исполняемым наш entrypoint скрипт
COPY ./entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

# 9. Открываем порт
EXPOSE 80

# 10. Указываем наш скрипт как точку входа
ENTRYPOINT ["/entrypoint.sh"]

# 11. Запускаем Supervisor через CMD.
# Эта команда будет передана в `exec "$@"` в нашем entrypoint скрипте.
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
```

### **Итоговый Порядок запуска**

Теперь, когда вы выполните `docker-compose up`, произойдет следующее:
1. Docker Compose запустит сервис `db`.
2. После старта контейнера `db`, Compose запустит контейнер `app` (из-за `depends_on`).
3. Внутри контейнера `app` первым делом запустится `ENTRYPOINT` — наш скрипт `/entrypoint.sh`.
4. Скрипт войдет в цикл и будет ждать, пока порт `5432` на хосте `db` станет доступен.
5. Как только порт станет доступен, скрипт выполнит команду миграции `flask db upgrade`.
6. После успешного выполнения миграции, команда `exec "$@"` запустит `CMD` из Dockerfile, то есть `/usr/bin/supervisord …`.
7. Supervisor запустится и поднимет все ваши сервисы (Gunicorn, Nginx, Celery), которые теперь будут работать с уже обновленной базой данных.

Этот подход является надежным и общепринятым стандартом для решения подобных задач при контейнеризации приложений.