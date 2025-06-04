Вот пример простого веб-сервера на Python с использованием только стандартной библиотеки, который возвращает JSON:

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
import json
from urllib.parse import urlparse

class JSONRequestHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        # Парсим URL для определения пути
        parsed_path = urlparse(self.path)
        
        # Устанавливаем заголовки ответа
        self.send_response(200)
        self.send_header('Content-Type', 'application/json')
        self.end_headers()
        
        # Формируем данные для ответа
        if parsed_path.path == '/':
            response_data = {
                'message': 'Hello, World!',
                'status': 'success',
                'path': self.path
            }
        else:
            response_data = {
                'error': 'Not Found',
                'status': 'fail',
                'path': self.path
            }
            self.send_response(404)
        
        # Конвертируем данные в JSON и отправляем
        self.wfile.write(json.dumps(response_data).encode())

def run(server_class=HTTPServer, handler_class=JSONRequestHandler, port=8000):
    server_address = ('', port)
    httpd = server_class(server_address, handler_class)
    print(f'Starting server on port {port}...')
    httpd.serve_forever()

if __name__ == '__main__':
    run()
```

Этот сервер:
1. Создает обработчик запросов, наследующий от BaseHTTPRequestHandler
2. Возвращает JSON-ответ для GET-запросов
3. По корневому пути '/' возвращает JSON с сообщением
4. Для других путей возвращает 404 ошибку в JSON-формате
5. Использует стандартные модули: http.server, json и urllib.parse

Для тестирования:
1. Запустите скрипт
2. Откройте в браузере http://localhost:8000/
3. Или используйте curl: `curl http://localhost:8000/`

Пример ответа для корневого пути:
```json
{
    "message": "Hello, World!",
    "status": "success",
    "path": "/"
}
```

Пример ответа для несуществующего пути:
```json
{
    "error": "Not Found",
    "status": "fail",
    "path": "/unknown"
}
```

Вы можете модифицировать словарь response_data, чтобы включать любые данные, которые вам нужно вернуть в JSON-формате.