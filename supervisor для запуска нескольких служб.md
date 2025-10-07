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

# **Шаг 3: Dockerfile**

Теперь соберем все это вместе в `Dockerfile`.
```Dockerfile
# 1. Выбираем базовый образ
FROM python:3.9-slim

# 2. Устанавливаем переменные окружения
ENV PYTHONUNBUFFERED 1

# 3. Устанавливаем необходимые пакеты: supervisor, nginx и другие
RUN apt-get update && apt-get install -y \
    supervisor \
    nginx \
    curl \
    && rm -rf /var/lib/apt/lists/*

# 4. Устанавливаем Python-зависимости
RUN pip install flask gunicorn

# 5. Копируем код приложения
COPY ./app /app

# 6. Копируем конфигурационные файлы
COPY ./supervisor/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY ./nginx/nginx.conf /etc/nginx/nginx.conf

# 7. Создаем директории для логов, указанные в supervisord.conf
RUN mkdir -p /var/log/gunicorn /var/log/nginx /var/log/celery

# 8. Открываем порт, который слушает Nginx
EXPOSE 80

# 9. Запускаем Supervisor!
# Это будет основной процесс (PID 1) в контейнере.
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
```

**Разбор `Dockerfile`:**

1. Мы используем базовый образ с Python.
2. Устанавливаем системные пакеты `supervisor` и `nginx`.
3. Устанавливаем Python-библиотеки.
4. Копируем наше приложение и файлы конфигурации в нужные места внутри образа.
5. **Важно:** Копируем `supervisord.conf` в `/etc/supervisor/conf.d/`. Supervisor автоматически подхватывает конфигурационные файлы из этой директории.
6. Создаем папки для логов.
7. `EXPOSE 80` информирует Docker, что контейнер будет слушать этот порт.
8. `CMD ["/usr/bin/supervisord", …]` — это команда, которая запускается при старте контейнера. Она запускает Supervisor, который, в свою очередь, запускает и контролирует все наши службы (Gunicorn, Nginx, Celery) в соответствии с файлом конфигурации.

# **Шаг 4: Сборка И Запуск контейнера**

1. Перейдите в корневую директорию проекта (`my-multi-service-app`).
2. **Соберите образ:**
```Bash
docker build -t my-multi-service-app .
```
3. **Запустите контейнер:**
```Bash
docker run -d -p 8080:80 --name my-app-instance my-multi-service-app
```
- `-d` — запуск в фоновом режиме.
- `-p 8080:80` — проброс порта 8080 вашего компьютера на порт 80 внутри контейнера.
- `--name` — удобное имя для контейнера.

# **Шаг 5: Проверка**
1. **Проверьте веб-приложение:** Откройте в браузере `http://localhost:8080`. Вы должны увидеть сообщение "Привет от Flask, Gunicorn и Nginx!".
2. **Посмотрите логи внутри контейнера:** Вы можете подключиться к контейнеру и посмотреть, что происходит.
```Bash
docker exec -it my-app-instance bash
```
    
Внутри контейнера можно посмотреть логи:
```Bash
tail -f /var/log/gunicorn.out.log
tail -f /var/log/nginx.out.log
tail -f /var/log/celery.out.log
```

Или использовать утилиту `supervisorctl`:
```Bash
supervisorctl status
```

Вывод будет примерно таким:
```
celery                           RUNNING   pid 10, uptime 0:05:12
gunicorn                         RUNNING   pid 8, uptime 0:05:12
nginx                            RUNNING   pid 9, uptime 0:05:12
```


Этот пример наглядно демонстрирует, как Supervisor действует в роли "менеджера процессов" внутри контейнера, позволяя вам запускать и контролировать несколько служб как единое целое.