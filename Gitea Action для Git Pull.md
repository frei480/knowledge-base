Настроить простой CI/CD: при пуше в локальный сервер **Gitea** автоматически будет обновлять репозиторий на другой Linux-машине. Это вполне нормально делается через **Gitea Actions**.
# Вариант 1: Через Gitea Actions (рекомендуется)
Gitea сам запускает job, который подключается к твоей Linux-машине и делает `git pull`.
## 1. Подготовь Linux-машину

Убедись, что:
- установлен git
- есть доступ по SSH
- репозиторий уже клонирован
```bash
cd /path/to/project
git clone http://your-gitea/repo.git
```

---
## 2. Настрой SSH Доступ Без Пароля

На Gitea runner (или сервере):
```bash
ssh-keygen -t ed25519
ssh-copy-id user@target-linux
```
Проверь:
```bash
ssh user@target-linux
```
---
## 3. Добавь Секрет В Gitea
В репозитории:
```
Settings → Secrets → Actions
```
Добавь:
```
SSH_PRIVATE_KEY
```
---
## 4. Создай Workflow
`.gitea/workflows/deploy.yml`
```yaml
name: Deploy

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: linux

    steps:
      - name: Setup SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_ed25519
          chmod 600 ~/.ssh/id_ed25519
          ssh-keyscan your-linux-host >> ~/.ssh/known_hosts

      - name: Pull repo on server
        run: |
          ssh user@your-linux-host "cd /path/to/project && git pull"
```

---

# Вариант 2: Через Webhook (проще, Но Менее гибко)

Gitea может дергать HTTP endpoint.

## 1. Подними На Linux Маленький Сервер

Например на Python:

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
import os

class Handler(BaseHTTPRequestHandler):
    def do_POST(self):
        os.system("cd /path/to/project && git pull")
        self.send_response(200)
        self.end_headers()

HTTPServer(('0.0.0.0', 8000), Handler).serve_forever()
```

---

## 2. В Gitea Настрой Webhook

```
Settings → Webhooks → Add Webhook
```

URL:

```
http://your-linux:8000
```

---
Чтобы гарантированно синхронизироваться, вместо `git pull` лучше:
```bash
git fetch --all
git reset --hard origin/main
```
