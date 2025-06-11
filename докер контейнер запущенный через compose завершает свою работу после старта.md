---
tags:
  - docker
---
Чтобы контейнер продолжал работать после запуска службы `aksusbd`, нужно изменить команду запуска так, чтобы основной процесс контейнера не завершался. Вот несколько вариантов:

# 1. Использовать `tail -f /dev/null` Для Вечного Ожидания
```yaml
command: sh -c "service aksusbd start && tail -f /dev/null"
```

# 2. Запустить Службу В Foreground (если поддерживается)
Если `aksusbd` может работать в foreground (например, с флагом `-F`):
```yaml
command: /usr/sbin/aksusbd -F
```

# 3. Использовать `sleep infinity`
```yaml
command: sh -c "service aksusbd start && sleep infinity"
```

# 4. Запуск Через `supervisord`
Если служба требует сложной настройки, используйте демон-менеджер:
1. Создайте `supervisord.conf`:
   ```ini
   [supervisord]
   nodaemon=true

   [program:aksusbd]
   command=service aksusbd start
   autostart=true
   ```
2. В Dockerfile:
   ```Dockerfile
   RUN apt-get update && apt-get install -y supervisor
   COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
   CMD ["/usr/bin/supervisord"]
   ```

# Пример Для `docker-compose.yml`
```yaml
version: '3'
services:
  your_service:
    image: your_image
    command: sh -c "service aksusbd start && tail -f /dev/null"
```

# Почему Это Работает?
- Docker контейнер работает до тех пор, пока активен основной процесс (PID 1).
- `tail -f /dev/null` или `sleep infinity` создают бесконечный процесс, который не завершается.
- Если служба может работать в foreground, лучше использовать этот режим.