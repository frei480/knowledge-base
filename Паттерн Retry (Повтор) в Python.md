---
tags:
  - python
Тип: Пересказ видео
Ссылка: https://www.youtube.com/watch?v=43vH_lnM7x0
Автор: Arjan Egges
---
# Паттерн Retry (Повтор) В Python

## 1. Введение: Проблема Временных Ошибок (Transient Failures)
Современные системы зависят от внешних сервисов: API, баз данных, микросервисов и LLM (ChatGPT и др.). Даже если ваш код идеален, эти сервисы могут временно давать сбой из-за:
* Сетевых задержек.
* Превышения лимитов запросов (Rate limiting).
* Кратковременной перегрузки сервера.

**Паттерн Retry** — это механизм, который оборачивает потенциально нестабильную операцию и автоматически повторяет её несколько раз, прежде чем окончательно выбросить ошибку.

---

## 2. Простая Реализация Retry
Самый примитивный способ — использование цикла `for` и блока `try…except`.

```python
def retry(operation: Callable[[], T], retries: int = 3, delay: float = 1.0) -> T:
    for attempt in range(1, retries + 1):
        try:
            return operation()
        except Exception as e:
            if attempt == retries:
                raise e # Если попытки исчерпаны, выбрасываем ошибку
            time.sleep(delay)
```
*Проблема:* Фиксированная задержка (delay) может быть неэффективной или даже вредной (создавать «шторм запросов» на сервер).

---

## 3. Экспоненциальная Задержка (Exponential Backoff)
Чтобы не перегружать сервер, который и так испытывает трудности, время ожидания между попытками должно увеличиваться.

**Формула:** `sleep_time = delay * (backoff ** (attempt - 1))`

```python
def retry(
    operation: Callable[[], T], 
    retries: int = 3, 
    delay: float = 1.0, 
    backoff: float = 2.0
) -> T:
    for attempt in range(1, retries + 1):
        try:
            return operation()
        except Exception as e:
            if attempt == retries:
                raise e
            sleep_time = delay * (backoff ** (attempt - 1))
            time.sleep(sleep_time)
```
*Результат:* Попытки будут происходить через 1с, 2с, 4с и т.д.

---

## 4. Паттерн «Декоратор» Для Retry
Чтобы сделать код чище и не оборачивать каждый вызов вручную, удобно использовать декораторы. Важно использовать `functools.wraps`, чтобы сохранить метаданные (имя, документацию) исходной функции.

```python
from functools import wraps

def retry_decorator(retries: int = 3, delay: float = 1.0, backoff: float = 2.0):
    def decorator(func: Callable[..., T]) -> Callable[..., T]:
        @wraps(func)
        def wrapper(*args, **kwargs) -> T:
            # Логика повторов здесь...
            return func(*args, **kwargs)
        return wrapper
    return decorator

@retry_decorator(retries=5)
def fetch_data():
    # нестабильный код
    pass
```

---

## 5. Стратегия Fallback (Резервный путь)
Иногда повтор одной и той же операции бесполезен (например, основной сервер полностью лежит). В таком случае можно использовать **Fallback** — вызов альтернативной функции.

**Пример:** Если основной API шуток недоступен, берем шутку из локального кэша или другого API.

```python
@retry_decorator(fallback_func=fetch_backup_joke)
def fetch_joke():
    # вызов основного API
```

### Продвинутый Подход: Список Операций
Можно передавать список функций, которые будут пробоваться по очереди:
```python
def retry(operations: list[Callable[[], T]]) -> T:
    for op in operations:
        try:
            return op()
        except Exception:
            continue
    raise RuntimeError("Все методы провалены")
```

---

## 6. Использование Готовых Библиотек (Tenacity)
В реальных проектах не стоит изобретать велосипед. Библиотека `tenacity` — стандарт индустрии для Retry в Python.

```python
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(5), 
    wait=wait_exponential(multiplier=1, min=2, max=10)
)
def unstable_function():
    # Код с автоматическим повтором
```

---

## 7. Когда Использовать И Когда Избегать
### Использовать:
* Сетевые запросы (HTTP, DB).
* Взаимодействие с LLM (некорректный JSON в ответе).
* Ошибки типа `RateLimitError`.

### Избегать:
* **Постоянные ошибки:** Неверный API-ключ, ошибка валидации (400 Bad Request) — повтор тут не поможет.
* **Побочные эффекты (Side Effects):** Если функция уже изменила данные в БД перед сбоем, повторный вызов может привести к дубликатам (проблема идемпотентности).
* **Риск «шторма повторов»:** Если тысячи клиентов одновременно начнут делать Retry, они «добьют» упавший сервер.

## Заключение
Retry Pattern делает софт устойчивым (robust). Начинайте с простых циклов, переходите к декораторам для чистоты кода и используйте `tenacity` для сложных сценариев в продакшене.