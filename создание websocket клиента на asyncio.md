---
tags:
  - python
---
Вот пример WebSocket клиента с использованием библиотеки `asyncio` и `websockets`:

```python
import asyncio
import websockets

async def websocket_client():
    uri = "wss://echo.websocket.org"  # Пример эхо-сервера для тестирования
    async with websockets.connect(uri) as websocket:
        # Отправка сообщения
        message = "Привет, WebSocket!"
        await websocket.send(message)
        print(f">>> Отправлено: {message}")

        # Получение ответа
        response = await websocket.recv()
        print(f"<<< Получено: {response}")

async def interactive_client():
    uri = "ws://localhost:8765"  # Локальный сервер для примера
    async with websockets.connect(uri) as websocket:
        # Запуск двух параллельных задач
        await asyncio.gather(
            receive_messages(websocket),
            send_messages(websocket)
        )

async def receive_messages(websocket):
    try:
        async for message in websocket:
            print(f"<<< Получено: {message}")
    except websockets.ConnectionClosed:
        print("Соединение закрыто сервером")

async def send_messages(websocket):
    try:
        while True:
            message = await asyncio.get_event_loop().run_in_executor(
                None, input, ">>> Введите сообщение: "
            )
            await websocket.send(message)
    except websockets.ConnectionClosed:
        print("Не удалось отправить сообщение: соединение закрыто")

if __name__ == "__main__":
    # Для простого примера
    # asyncio.run(websocket_client())
    
    # Для интерактивного клиента
    asyncio.run(interactive_client())
```

# Пояснение:
1. **Установка зависимостей**:
   ```bash
   pip install websockets
   ```

2. **Простой клиент (`websocket_client`)**:
   - Подключается к эхо-серверу
   - Отправляет одно сообщение
   - Получает и выводит ответ

3. **Интерактивный клиент (`interactive_client`)**:
   - Поддерживает одновременную отправку и получение сообщений
   - Использует `asyncio.gather` для параллельного выполнения задач
   - `receive_messages`: бесконечно слушает сервер
   - `send_messages`: читает ввод пользователя и отправляет сообщения

4. **Обработка ошибок**:
   - Автоматически обрабатывает закрытие соединения
   - Ловит исключение `ConnectionClosed`

# Особенности:
- Используйте `wss://` для защищенных соединений
- Для работы с CLI вводом используется `run_in_executor`, чтобы не блокировать event loop
- Для тестирования можно использовать [официальный эхо-сервер](wss://echo.websocket.org) или запустить локальный сервер

Пример сервера для тестирования (требует `websockets`):
```python
# server.py
import asyncio
import websockets

async def echo(websocket):
    async for message in websocket:
        await websocket.send(f"Эхо: {message}")

async def main():
    async with websockets.serve(echo, "localhost", 8765):
        await asyncio.Future()  # Бесконечное выполнение

asyncio.run(main())
```

Запустите сервер и клиент в разных терминалах для тестирования интерактивного режима.
[[asyncio run_in_executor]]
