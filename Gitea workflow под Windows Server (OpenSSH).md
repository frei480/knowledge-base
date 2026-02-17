
# 1️⃣ Что Нам Понадобится

- SSH-доступ к серверу (ключ!)
- В репозитории настроенные Secrets:
    - `SSH_HOST`
    - `SSH_USER`
    - `SSH_KEY` (приватный ключ)
    - `SSH_PORT` (обычно 22)
    - `DEPLOY_PATH` (куда копировать на сервере)

Gitea → **Settings → Actions → Secrets**

---
# 2️⃣ Главная Идея

В Gitea Actions доступны SHA коммитов:
- `github.sha` — текущий коммит
- `github.event.before` — предыдущий

Значит можно:
```bash
git diff --name-only OLD NEW
```
и получить список изменённых файлов.
# 3️⃣ Workflow (готовый)

Создай файл:
```sh
.gitea/workflows/deploy.yaml
```

Linux-сервер → `scp` + `mkdir -p` + `rm` работает
Windows + OpenSSH →
- нет `mkdir -p`
- нет `rm -f`
- пути `\`
- shell = **PowerShell**, а не bash

Значит:
👉 команды нужно отправлять **PowerShell-совместимые**

---

# Что Меняется Принципиально

Мы НЕ меняем идею workflow.
Мы меняем **удалённые команды**:

|Linux|Windows|
|---|---|
|mkdir -p dir|`New-Item -ItemType Directory -Force`|
|rm -f file|`Remove-Item -Force`|
|/var/www/app|`C:\inetpub\app`|
|$(dirname file)|PowerShell Split-Path|

---

# ВАЖНО: Путь В Secrets
```
DEPLOY_PATH = C:\deploy\myapp
```
(без кавычек)
---

# Готовый Gitea Workflow Под Windows Server (OpenSSH)

```yaml
name: Deploy to Windows via SSH

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan -p ${{ secrets.SSH_PORT }} ${{ secrets.SSH_HOST }} >> ~/.ssh/known_hosts

      - name: Get changes
        run: |
          BEFORE="${{ gitea.event.before }}"
          AFTER="${{ gitea.sha }}"

          if [ "$BEFORE" = "0000000000000000000000000000000000000000" ]; then
            git ls-files > changes.txt
            sed -i 's/^/A /' changes.txt
          else
            git diff --name-status $BEFORE $AFTER > changes.txt
          fi

          cat changes.txt

      - name: Sync to Windows
        run: |
          while read status file; do
            # Windows path
            winfile=$(echo "$file" | sed 's/\//\\/g')
            if [ "$status" = "D" ]; then
              echo "Deleting $file"
              ssh -p ${{ secrets.SSH_PORT }} ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }} powershell -Command "
                \$path='${{ secrets.DEPLOY_PATH }}\\$winfile';
                if (Test-Path \$path) { Remove-Item \$path -Force }
              "
            else
              echo "Uploading $file"
              ssh -p ${{ secrets.SSH_PORT }} ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }} powershell -Command "
                \$dir=Split-Path '${{ secrets.DEPLOY_PATH }}\\$winfile';
                New-Item -ItemType Directory -Force -Path \$dir | Out-Null
              "
              scp -P ${{ secrets.SSH_PORT }} "$file" \
                ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }}:"${{ secrets.DEPLOY_PATH }}/$file"
            fi

          done < changes.txt
```

---

# Почему Scp Оставляем С `/` А Не `\`

Это ОЧЕНЬ важный момент.
OpenSSH на Windows:
- SCP принимает `/` (unix-стиль)
- PowerShell требует `\`

Поэтому:
- команды → `\`
- scp → `/`

Иначе будет:
```sh
scp: ambiguous target
scp: No such file or directory
```

---

# Типичные Проблемы (90% Людей ловят)

## ❌ Права На Ключ

Windows OpenSSH требует:
```powershell
icacls C:\Users\user\.ssh\authorized_keys /inheritance:r
icacls C:\Users\user\.ssh\authorized_keys /grant user:F
```

## ❌ Shell Не PowerShell

Проверь на сервере:
```
C:\ProgramData\ssh\sshd_config
```

должно быть:
```
Subsystem powershell C:/Windows/System32/WindowsPowerShell/v1.0/powershell.exe -sshs -NoLogo -NoProfile
```

перезапуск:
```
Restart-Service sshd
```

---

# Результат

Теперь у тебя:
- 🧠 только изменённые файлы
- 🪟 Windows сервер
- 🔐 через SSH
- 🧹 удаление файлов
- ⚡ быстрый incremental deploy

