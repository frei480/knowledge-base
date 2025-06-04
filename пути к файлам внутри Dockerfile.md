---
tags:
  - docker
---
пусть есть Dockerfile:
```
FROM debian:bookworm-slim
WORKDIR /app
COPY HASP_Linux . 
RUN chmod +x ./HASP_Linux/aksusbd/dinst
``` 

выдает ошибку RUN chmod +x ./HASP_Linux/aksusbd/dinst: 
0.162 chmod: cannot access './HASP_Linux/aksusbd/dinst': No such file or directory

Ошибка возникает из-за некорректного пути к файлу `dinst`. Вот исправленный Dockerfile:

```dockerfile
FROM debian:bookworm-slim
WORKDIR /app
COPY HASP_Linux/aksusbd/dinst ./
RUN chmod +x ./dinst
```

**Пояснение:**

1. Если `HASP_Linux` это **директория** на хосте с файлом `aksusbd/dinst`, то:
   - `COPY HASP_Linux .` копирует **содержимое** директории в `/app`, поэтому файл будет находиться в `/app/aksusbd/dinst`.
   - Ошибка в пути: в вашей версии вы обращаетесь к `./HASP_Linux/aksusbd/dinst`, но после копирования структура будет `/app/aksusbd/dinst`.

2. Если вы хотите сохранить структуру `HASP_Linux/aksusbd/dinst` внутри контейнера, измените команду копирования:
   ```dockerfile
   COPY HASP_Linux ./HASP_Linux
   ```
   Тогда путь к файлу будет `./HASP_Linux/aksusbd/dinst`.

**Проверьте:**
- Убедитесь, что файл `dinst` действительно существует в вашей локальной директории `HASP_Linux/aksusbd/`.
- Если `HASP_Linux` это архив или файл, а не директория — его нужно распаковать с помощью `ADD` или отдельной команды.