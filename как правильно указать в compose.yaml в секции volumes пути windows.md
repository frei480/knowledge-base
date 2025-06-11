---
tags:
  - docker
---
Для указания путей Windows в секции `volumes` файла `docker-compose.yaml` нужно учитывать особенности синтаксиса и преобразование путей. Вот основные варианты:

---

### 1. **Локальный путь (Windows-стиль):**
Если файлы находятся на локальном диске (например, `C:\myapp\data`):
```yaml
volumes:
  - "C:/myapp/data:/container/path"
```
**Важно:**
- Используйте **прямые слеши** `/` вместо обратных `\`.
- Путь в Windows **должен существовать** на вашем компьютере.
- Если путь содержит пробелы — заключите его в **двойные кавычки**:
  ```yaml
  - "C:/Program Files/myapp:/app"
  ```

---

### 2. **UNC-пути (для сетевых ресурсов или WSL2):**
Для длинных путей или работы с WSL2 используйте формат с двойным слешем:
```yaml
volumes:
  - "//C/myapp/data:/container/path"
```

---

### 3. **Относительные пути:**
Если путь указан относительно директории с `docker-compose.yaml`:
```yaml
volumes:
  - "./data:/app/data"
```

---

### 4. **Проверка монтирования:**
Чтобы убедиться, что том смонтирован правильно:
```bash
docker compose exec <service-name> ls /container/path
```

---

### Пример полного `compose.yaml`:
```yaml
version: "3.8"

services:
  myapp:
    image: debian:bookworm-slim
    volumes:
      - "C:/Projects/myapp/config:/app/config"  # Абсолютный путь
      - "./data:/app/data"                      # Относительный путь
      - "//C/Users/Admin/Documents:/docs"       # UNC-путь
```

---

### Частые ошибки:
1. **Обратные слеши** (`\`):
   ```yaml
   # Неправильно!
   volumes:
     - "C:\myapp\data:/app/data"
   ```

2. **Пробелы без кавычек**:
   ```yaml
   # Неправильно!
   volumes:
     - C:/Program Files/myapp:/app
   ```

3. **Несуществующий путь** на хосте — Docker создаст папку, но это может привести к неожиданному поведению.

---

Если вы используете **Docker Desktop с WSL2**, пути к файлам в подсистеме Linux монтируются через `/mnt` (например, `/mnt/c/Users/…`), но в `docker-compose.yaml` лучше использовать нативные Windows-пути, как показано выше.