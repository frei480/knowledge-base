---
tags:
  - python
Тип: Пересказ видео
Ссылка: https://www.youtube.com/watch?v=GMBiCMsEsq8
Автор: Arjan Egges
---
# Путь к Production-Ready приложению

## Intro
Проблема: Большинство туториалов показывают "счастливый путь", который ломается при первой же нагрузке или некорректном вводе. Цель лекции — взять работающий скрипт конвертации валют и сделать его надежным.

## Что такое "Production-Ready" на самом деле?
Код готов к эксплуатации (production), если он:
*   **Наблюдаем (Observable):** Мы знаем, что происходит внутри (логи).
*   **Безопасен:** Защищен от инъекций и перегрузок.
*   **Поддерживаем:** Логика отделена от инфраструктуры.
*   **Устойчив:** Он не падает от отсутствия данных, а корректно отвечает.

---

## Шаг 1: Правильные типы (Decimal вместо float)
Для финансовых операций `float` — это катастрофа из-за ошибок округления двоичной системы.
**Решение:** Используем стандартную библиотеку `decimal`.

```python
# service.py
from decimal import Decimal

def convert(self, from_currency: str, to_currency: str, amount: Decimal) -> Decimal:
    # Использование Decimal гарантирует точность до последнего знака
    rate = self.get_rate(from_currency, to_currency)
    return amount * rate
```

---

## Шаг 2: Валидация входных данных
FastAPI позволяет проверять данные на уровне эндпоинта. Мы должны ограничить длину кодов валют (строго 3 символа) и сумму (больше 0).

```python
# api.py
from fastapi import Query
from typing import Annotated

@router.get("/convert")
def convert(
    from_currency: Annotated[str, Query(min_length=3, max_length=3)],
    to_currency: Annotated[str, Query(min_length=3, max_length=3)],
    amount: Annotated[Decimal, Query(gt=0)]
):
    ...
```

---

## Шаг 3: Вынос бизнес-логики в Сервис
Эндпоинт должен только принимать запрос и возвращать ответ. Вся "математика" и работа с данными уходит в `Service Layer`. Это позволяет тестировать логику без запуска веб-сервера.

```python
# service.py
class ExchangeRateService:
    def __init__(self, db: Session):
        self.db = db

    def convert(self, from_curr: str, to_curr: str, amount: Decimal):
        # Вся магия здесь
        ...

# api.py
@router.get("/convert")
def convert_endpoint(
    amount: Decimal,
    # Внедрение зависимости (Dependency Injection)
    service: ExchangeRateService = Depends(get_exchange_rate_service)
):
    return service.convert(..., amount)
```

---

## Шаг 4: Добавление персистентности (БД)
Промышленные данные живут в БД, а не в словарях Python. Используем SQLAlchemy 2.0.

```python
# models.py
from sqlalchemy.orm import Mapped, mapped_column

class Conversion(Base):
    __tablename__ = "conversions"
    id: Mapped[int] = mapped_column(primary_key=True)
    from_currency: Mapped[str] = mapped_column(fixed_length=3)
    amount: Mapped[Decimal]
    result: Mapped[Decimal]
    timestamp: Mapped[datetime] = mapped_column(default=datetime.utcnow)
```

---

## Шаг 5: Health Check
Системы мониторинга (Docker, Kubernetes) должны знать, что сервер не просто запущен, но и "здоров".

```python
# api.py
@router.get("/health")
def health_check():
    # Можно добавить проверку соединения с БД
    return {"status": "ok", "version": "1.0.0"}
```

---

## Шаг 6: Защитное программирование и обработка ошибок
Программа не должна выдавать `Internal Server Error (500)`. Если курса валют нет, нужно вернуть `404`.

```python
# service.py
if not rate_entry:
    logger.warning(f"Курс {from_curr} -> {to_curr} не найден")
    raise HTTPException(
        status_code=404, 
        detail=f"Exchange rate from {from_curr} to {to_curr} not found."
    )
```

---

## Шаг 7: Управление конфигурацией
Используем `pydantic-settings` для подгрузки настроек из `.env` файлов или переменных окружения.

```python
# config.py
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    database_url: str = "sqlite:///./db.sqlite3"
    api_key: str  # Обязательная переменная
    model_config = SettingsConfigDict(env_file=".env")

settings = Settings()
```

---

## Шаг 8: Ограничение частоты запросов (Rate Limiting)
Защита от ботов и чрезмерной нагрузки. Используем библиотеку `slowapi`.

```python
# api.py
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@router.get("/convert")
@limiter.limit("5/minute") # 5 запросов в минуту на один IP
def convert(request: Request, ...):
    ...
```

---

## Шаг 9: Написание тестов
Используем `pytest` и `TestClient`. Важно тестировать как успешные случаи, так и ошибки.

```python
# test_api.py
def test_convert_missing_rate(client):
    # Пытаемся конвертировать несуществующую валюту
    response = client.get("/convert?from_currency=XYZ&to_currency=USD&amount=10")
    assert response.status_code == 404
    assert "not found" in response.json()["detail"]
```

---

## Шаг 10: Мониторинг и Логирование
Заменяем `print()` на `logging`. Логи в продакшене — это единственный способ понять, почему ночью всё сломалось.

```python
import logging

logger = logging.getLogger(__name__)
logging.basicConfig(level=logging.INFO)

logger.info("Application started")
logger.error("DB Connection failed")
```

---

## Шаг 11: Deployment (Docker + GitHub Actions)
Упаковываем всё в контейнер, чтобы приложение работало одинаково везде.

```dockerfile
# Dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
# Не запускаем от root для безопасности!
USER 1000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Финальные мысли
Production-ready — это не конечная точка, а процесс. Начав с типов данных и закончив Docker-контейнером, вы создаете систему, за которую не стыдно перед пользователями и коллегами-девопсами.