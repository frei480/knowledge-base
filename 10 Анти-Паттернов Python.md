---
tags:
  - python
  - АрхитектураПО
Автор: Arjan Egges
Ссылка: https://www.youtube.com/watch?v=ts38mSIUPSg
Тип: Пересказ видео
Название: 10 Анти-Паттернов Python
---
00:00 Введение

• Некоторые шаблоны кода в Python могут вредить проекту.
• Видео посвящено десяти анти-паттернам Python.

# Анти-паттерн 1 - Использование Исключений Для Потока Управления

• Пример скрипта, извлекающего данные из API.
```python
import random
import time


# Simulated API call that randomly succeeds or times out
def fetch_from_primary_api(city: str) -> dict[str, str]:
    if simulate_timeout():
        raise TimeoutError("Primary API timed out.")
    return {"status": "success", "data": f"Weather in {city} is sunny."}


def fetch_from_backup_api(city: str) -> dict[str, str]:
    if simulate_timeout():
        raise TimeoutError("Backup API timed out.")
    return {"status": "success", "data": f"Backup: Weather in {city} is cloudy."}


def simulate_timeout() -> bool:
    time.sleep(0.2)  # network delay
    return random.random() < 0.5


# Bad: using exceptions for expected control flow
def get_weather_forecast(city: str) -> dict[str, str]:
    try:
        return fetch_from_primary_api(city)
    except TimeoutError:
        try:
            return fetch_from_backup_api(city)
        except TimeoutError:
            return {"status": "error", "message": "Both APIs timed out."}


def main():
    result = get_weather_forecast("New York")
    print(result)


if __name__ == "__main__":
    main()
```
• Функция выборки из основного API и функция имитации тайм-аута.
• Проблема в функции `get_weather_forecast()` получения прогноза погоды, которая использует исключения для управления потоком.

01:55 Проблемы с исключениями

• Исключения дороги с точки зрения производительности.
• Много типов исключений делает код сложным для чтения.
• Решение: заменить исключения на возвращаемые значения и реорганизовать код.
```python
import random
import time


# Simulated API call that randomly succeeds or times out
def fetch_from_primary_api(city: str) -> dict[str, str]:
    if simulate_timeout():
        return {"status": "error", "message": "Primary API timed out."}
    return {"status": "success", "data": f"Weather in {city} is sunny."}


def fetch_from_backup_api(city: str) -> dict[str, str]:
    if simulate_timeout():
        return {"status": "error", "message": "Backup API timed out."}
    return {"status": "success", "data": f"Backup: Weather in {city} is cloudy."}


def simulate_timeout() -> bool:
    time.sleep(0.2)  # network delay
    return random.random() < 0.5


# Good: using return values for expected control flow
def get_weather_forecast(city: str) -> dict[str, str]:
    result = fetch_from_primary_api(city)
    if result["status"] == "error":
        result = fetch_from_backup_api(city)
    if result["status"] == "error":
        return {"status": "error", "message": "Both APIs timed out."}
    return result


def main():
    result = get_weather_forecast("New York")
    print(result)


if __name__ == "__main__":
    main()
```
# Анти-паттерн 2 - Использование Классов Только Со Статическими Методами

• Пример класса с множеством статических методов.
```python
import numpy as np
import pandas as pd


class DataPreprocessingUtils:
    @staticmethod
    def fill_missing_values(df: pd.DataFrame) -> pd.DataFrame:
        """Fill missing numeric values with the median of each column."""
        return df.fillna(df.median(numeric_only=True))

    @staticmethod
    def normalize_columns(df: pd.DataFrame, columns: list[str]) -> pd.DataFrame:
        """Min-max normalize specified columns in the DataFrame."""
        df = df.copy()
        for col in columns:
            min_val = df[col].min()
            max_val = df[col].max()
            if min_val == max_val:
                df[col] = 0.0  # avoid division by zero
            else:
                df[col] = (df[col] - min_val) / (max_val - min_val)
        return df

    @staticmethod
    def encode_categorical(df: pd.DataFrame, columns: list[str]) -> pd.DataFrame:
        """Convert categorical columns into one-hot encoded columns."""
        return pd.get_dummies(df, columns=columns, drop_first=True)


def main() -> None:
    df = pd.DataFrame(
        {
            "age": [25, 30, np.nan, 22],
            "income": [50000, 60000, 55000, np.nan],
            "gender": ["male", "female", "female", "male"],
        }
    )

    df = DataPreprocessingUtils.fill_missing_values(df)
    df = DataPreprocessingUtils.normalize_columns(df, ["age", "income"])
    df = DataPreprocessingUtils.encode_categorical(df, ["gender"])

    print(df)


if __name__ == "__main__":
    main()
```
• В Python можно использовать модули с функциями вместо классов.
• Решение: использовать модули с функциями вместо классов.
```python
import numpy as np
import pandas as pd
from data_processing import encode_categorical, fill_missing_values, normalize_columns


def main() -> None:
    df = pd.DataFrame(
        {
            "age": [25, 30, np.nan, 22],
            "income": [50000, 60000, 55000, np.nan],
            "gender": ["male", "female", "female", "male"],
        }
    )

    df = fill_missing_values(df)
    df = normalize_columns(df, ["age", "income"])
    df = encode_categorical(df, ["gender"])

    print(df)


if __name__ == "__main__":
    main()
```
# Анти-паттерн 3 - Неожиданное Переопределение Методов Dunder

• Пример класса оплаты с переопределенным методом пожертвования.
```python
class Payment:
    def __new__(cls, payment_type: str):
        if payment_type == "paypal":
            return object.__new__(PaypalPayment)
        elif payment_type == "card":
            return object.__new__(StripePayment)
        else:
            raise ValueError(f"Unsupported payment type: {payment_type}")

    def pay(self, amount: int) -> None:
        raise NotImplementedError


class PaypalPayment(Payment):
    def pay(self, amount: int) -> None:
        print(f"Paying {amount} using Paypal")


class StripePayment(Payment):
    def pay(self, amount: int) -> None:
        print(f"Paying {amount} using Stripe")


def main() -> None:
    my_payment = Payment("card")
    my_payment.pay(100)


if __name__ == "__main__":
    main()
```
• Неожиданное поведение при вызове метода оплаты.
• Решение: использовать перечисления и функции оплаты вместо наследования.
```python
from enum import StrEnum
from typing import Callable


class PaymentMethod(StrEnum):
    PAYPAL = "paypal"
    CARD = "card"


type PaymentFn = Callable[[int], None]


def pay_paypal(amount: int) -> None:
    print(f"Paying {amount} using Paypal")


def pay_stripe(amount: int) -> None:
    print(f"Paying {amount} using Stripe")


def get_payment_method(payment_type: str) -> PaymentFn:
    if payment_type == PaymentMethod.PAYPAL:
        return pay_paypal
    elif payment_type == PaymentMethod.CARD:
        return pay_stripe
    else:
        raise ValueError(f"Unsupported payment type: {payment_type}")


PAYMENT_METHODS: dict[PaymentMethod, PaymentFn] = {
    PaymentMethod.CARD: pay_stripe,
    PaymentMethod.PAYPAL: pay_paypal,
}


def main():
    # using the dictionary
    my_payment_fn = PAYMENT_METHODS[PaymentMethod.PAYPAL]
    my_payment_fn(100)

    # using the function
    my_payment_fn = get_payment_method("card")
    my_payment_fn(100)


if __name__ == "__main__":
    main()
```
# Анти-паттерн 4 - Использование Жестко Закодированных Значений

• Пример приложения Streamlit, извлекающего данные из URL.
```python
import numpy as np
import pandas as pd
import streamlit as st

# Hardcoded constants
DATE_COLUMN = "date/time"
DATA_URL = (
    "https://s3-us-west-2.amazonaws.com/streamlit-demo-data/uber-raw-data-sep14.csv.gz"
)


@st.cache_data
def load_data(nrows: int) -> pd.DataFrame:
    data = pd.read_csv(DATA_URL, nrows=nrows)
    data.rename(lambda x: str(x).lower(), axis="columns", inplace=True)
    data[DATE_COLUMN] = pd.to_datetime(data[DATE_COLUMN])
    return data


# 🧨 All UI strings are hardcoded — even small changes are tedious
st.title("Uber pickups in NYC")

data_load_state = st.text("Loading data...")
data = load_data(10000)
data_load_state.text("Loading complete!")

if st.checkbox("Show raw data"):
    st.subheader("Raw data")
    st.write(data)

st.subheader("Number of pickups by hour")
hist_values = np.histogram(data[DATE_COLUMN].dt.hour, bins=24, range=(0, 24))[0]
st.bar_chart(hist_values)

hour_to_filter = st.slider("Hour", 0, 23, 17)
filtered_data = data[data[DATE_COLUMN].dt.hour == hour_to_filter]

st.subheader(f"Map of all pickups at {hour_to_filter}:00")
st.map(filtered_data)
```
• Использование жестко закодированных значений в коде.
• Решение: избегать жестко закодированных значений и использовать переменные.

10:14 Проблемы с жестко закодированными строками в Streamlit

• В Streamlit все строки пользовательского интерфейса жестко закодированы в вызовах.
• Это затрудняет текстовые исправления и поддержку нескольких языков.
• Локализованная версия от спонсора позволяет управлять переводами в одном месте.

10:46 Интеграция сервиса Localize

• Localize позволяет перенести весь текст в одно место и синхронизировать его с приложением.
• Установка соединения с Localize проста: нужно создать API-ключ и клиент в приложении.
• Вспомогательные функции помогают быстро получать переводы по ключу.

11:34 Использование локализованного переводчика

• Локализованный переводчик инициализирует клиента и извлекает переводы.
• Метод gettranslations извлекает переводы по идентификаторам.
• Переводы можно добавлять и переводить с помощью искусственного интеллекта прямо в интерфейсе.

12:23 Преимущества локализованного интерфейса

• Переводы с использованием искусственного интеллекта выполняются быстрее и точнее.
• Локализованный интерфейс упрощает рабочий процесс и обеспечивает точность переводов.
• Localize предоставляет информацию о прогрессе переводов и прогнозы их готовности.
```python
import io
import json
import os
import zipfile

import lokalise
import numpy as np
import pandas as pd
import requests
import streamlit as st
from dotenv import load_dotenv

load_dotenv()
LOKALISE_API_KEY = os.getenv("LOKALISE_API_KEY")
LOKALISE_PROJECT_ID = os.getenv("LOKALISE_PROJECT_ID")
LANGUAGE = "nl"


class LokaliseTranslator:
    def __init__(self, api_key: str, project_id: str, language: str) -> None:
        self.client = lokalise.Client(api_key)
        self.project_id = project_id
        self.language = language
        self.translations = self.get_translations()

    def get_translations(self) -> dict[str, str]:
        response = self.client.download_files(
            self.project_id,
            {"format": "json", "original_filenames": True, "replace_breaks": False},
        )
        translations_url = response["bundle_url"]

        # Download and extract the ZIP file
        zip_response = requests.get(translations_url)
        zip_file = zipfile.ZipFile(io.BytesIO(zip_response.content))

        # Find the JSON file corresponding to the selected language
        json_filename = f"{self.language}/no_filename.json"
        with zip_file.open(json_filename) as json_file:
            return json.load(json_file)

    def __call__(self, key: str) -> str:
        return self.translations.get(key, key)


translator = LokaliseTranslator(LOKALISE_API_KEY, LOKALISE_PROJECT_ID, LANGUAGE)

st.title(translator("dashboard_title"))

DATE_COLUMN = "date/time"
DATA_URL = (
    "https://s3-us-west-2.amazonaws.com/streamlit-demo-data/uber-raw-data-sep14.csv.gz"
)


@st.cache_data
def load_data(nrows):
    data = pd.read_csv(DATA_URL, nrows=nrows)
    data.rename(lambda x: str(x).lower(), axis="columns", inplace=True)
    data[DATE_COLUMN] = pd.to_datetime(data[DATE_COLUMN])
    return data


data_load_state = st.text(translator("loading_data"))
data = load_data(10000)
data_load_state.text(translator("done"))

if st.checkbox(translator("show_raw_data")):
    st.subheader(translator("raw_data"))
    st.write(data)

st.subheader(translator("nb_pickups_hour"))
hist_values = np.histogram(data[DATE_COLUMN].dt.hour, bins=24, range=(0, 24))[0]
st.bar_chart(hist_values)

# Some number in the range 0-23
hour_to_filter = st.slider("hour", 0, 23, 17)
filtered_data = data[data[DATE_COLUMN].dt.hour == hour_to_filter]

st.subheader(translator("map_all_pickups") % hour_to_filter)
st.map(filtered_data)
```

# Анти-паттерн 5: Использование Декораторов Для Внедрения Данных

• Декораторы могут быть сложными и запутанными, если их использовать для внедрения данных.
• Пример с декоратором `inject_config` показывает, как это может усложнить код.
```python
import json
from functools import wraps
from typing import Any, Callable


def load_config(file_name: str) -> dict[str, Any]:
    """Load configuration from the specified file."""
    with open(file_name, "r") as f:
        return json.load(f)


def inject_config(file_name: str) -> Callable[[Callable[..., Any]], Callable[..., Any]]:
    def decorator(func: Callable[..., Any]) -> Callable[..., Any]:
        @wraps(func)
        def wrapper(*args: Any, **kwargs: Any) -> Any:
            config: dict[str, Any] = load_config(file_name)
            return func(config, *args, **kwargs)

        return wrapper

    return decorator


@inject_config("config.json")
def main(config: dict[str, Any]) -> None:
    print(f"Training with config: {config}")


if __name__ == "__main__":
    main()
```
• Простое решение: вызвать функцию для загрузки конфигурации напрямую.
```python
import json
from typing import Any


def load_config(file_name: str) -> dict[str, Any]:
    """Load configuration from the specified file."""
    with open(file_name, "r") as f:
        return json.load(f)


def main() -> None:
    config = load_config("config.json")
    print(f"Training with config: {config}")


if __name__ == "__main__":
    main()
```
# Анти-паттерн 6: Чрезмерная Разработка Шаблонов Проектирования

• Python динамичен и выразителен, поэтому не всегда нужны сложные шаблоны проектирования.
• Пример с экспортером данных показывает, что можно использовать функции вместо классов.
```python
import json
from abc import ABC, abstractmethod
from dataclasses import dataclass


@dataclass
class Report:
    title: str
    content: str


class Exporter(ABC):
    """Abstract base class for data exporters."""

    @abstractmethod
    def export(self, filename: str, report: Report) -> None:
        pass


class CSVExporter(Exporter):
    """Exporter for CSV format."""

    def export(self, filename: str, report: Report) -> None:
        csv_data = f"{report.title}, {report.content}\n"
        print("Exporting to CSV...")
        with open(filename, "w") as f:
            f.write(csv_data)
        print("Done.")


class JSONExporter(Exporter):
    """Exporter for JSON format."""

    def export(self, filename: str, report: Report) -> None:
        print("Exporting to JSON...")
        json_data = {
            "title": report.title,
            "content": report.content,
        }
        with open(filename, "w") as f:
            json.dump(json_data, f)
        print("Done.")


class ExporterFactory:
    """Factory for creating exporters based on format string."""

    @staticmethod
    def get_exporter(format: str) -> Exporter:
        if format == "csv":
            return CSVExporter()
        elif format == "json":
            return JSONExporter()
        else:
            raise ValueError(f"Unsupported export format: {format}")


def main() -> None:
    report = Report(
        title="Quarterly Earnings",
        content="Here are the earnings for the last quarter...",
    )

    exporter = ExporterFactory.get_exporter("json")
    exporter.export("report.json", report)


if __name__ == "__main__":
    main()
```
• Простое решение: использовать словарь для сопоставления форматов с функциями экспорта.
```python
import json
from dataclasses import dataclass
from typing import Callable


@dataclass
class Report:
    title: str
    content: str


type ExportFn = Callable[[str, Report], None]


def export_to_csv(filename: str, report: Report) -> None:
    print("Exporting to CSV...")
    csv_data = f"{report.title}, {report.content}\n"
    with open(filename, "w") as f:
        f.write(csv_data)
    print("Done.")


def export_to_json(filename: str, report: Report) -> None:
    print("Exporting to JSON...")
    json_data = {
        "title": report.title,
        "content": report.content,
    }
    with open(filename, "w") as f:
        json.dump(json_data, f)
    print("Done.")


EXPORTERS = {
    "csv": export_to_csv,
    "json": export_to_json,
}


def get_exporter(format: str) -> ExportFn:
    """Factory function to get the appropriate exporter function based on format string."""
    if format in EXPORTERS:
        return EXPORTERS[format]
    else:
        raise ValueError(f"Unsupported export format: {format}")


def main() -> None:
    report = Report(
        title="Quarterly Earnings",
        content="Here are the earnings for the last quarter...",
    )

    export_fn = get_exporter("json")
    export_fn("report.json", report)


if __name__ == "__main__":
    main()
```
# Анти-паттерн 7: Неуместной Интимности

• Экспортные функции должны знать структуру отчета, что усложняет код.
• Решение: перенести зависимости от структуры отчета в класс отчета.
• Добавить методы для представления отчета в разных форматах, чтобы функции экспорта могли работать с любыми объектами.
```python
import json
from dataclasses import dataclass
from typing import Callable


@dataclass
class Report:
    title: str
    content: str

    def to_csv(self) -> str:
        return f"{self.title}, {self.content}\n"

    def to_json(self) -> str:
        return json.dumps(
            {
                "title": self.title,
                "content": self.content,
            },
            indent=4,
        )


type ExportFn = Callable[[str, Report], None]


def export_to_csv(filename: str, report: Report) -> None:
    print("Exporting to CSV...")
    with open(filename, "w") as f:
        f.write(report.to_csv())
    print("Done.")


def export_to_json(filename: str, report: Report) -> None:
    print("Exporting to JSON...")
    with open(filename, "w") as f:
        json.dump(report.to_json(), f)
    print("Done.")


EXPORTERS = {
    "csv": export_to_csv,
    "json": export_to_json,
}


def get_exporter(format: str) -> ExportFn:
    """Factory function to get the appropriate exporter function based on format string."""
    if format in EXPORTERS:
        return EXPORTERS[format]
    else:
        raise ValueError(f"Unsupported export format: {format}")


def main() -> None:
    report = Report(
        title="Quarterly Earnings",
        content="Here are the earnings for the last quarter...",
    )

    export_fn = get_exporter("json")
    export_fn("report.json", report)


if __name__ == "__main__":
    main()
```
21:00 Введение в класс протокола

• Использование класса протокола для абстракции.
• Создание класса "экспортируемый" с методами преобразования в CSV и JSON.
• Изменение отчета для поддержки экспорта.
# Анти-паттерн 8: Слишком Много Прямой Связи
21:31 Применение класса протокола

• Функции больше не зависят от конкретного типа данных.
• Введение класса "бюджетный" с методами для CSV и JSON.
• Пример использования класса "бюджетный" для экспорта.
```python
import json
from dataclasses import dataclass
from typing import Callable, Protocol


@dataclass
class Report:
    title: str
    content: str

    def to_csv(self) -> str:
        return f"{self.title}, {self.content}\n"

    def to_json(self) -> str:
        return json.dumps(
            {
                "title": self.title,
                "content": self.content,
            },
            indent=4,
        )


@dataclass
class Budget:
    title: str
    amount: float

    def to_csv(self) -> str:
        return f"{self.title}, {self.amount}\n"

    def to_json(self) -> str:
        return json.dumps(
            {
                "title": self.title,
                "amount": self.amount,
            },
            indent=4,
        )


type ExportFn = Callable[[str, Exportable], None]


class Exportable(Protocol):
    def to_csv(self) -> str: ...

    def to_json(self) -> str: ...


def export_to_csv(filename: str, data: Exportable) -> None:
    print("Exporting to CSV...")
    with open(filename, "w") as f:
        f.write(data.to_csv())
    print("Done.")


def export_to_json(filename: str, data: Exportable) -> None:
    print("Exporting to JSON...")
    with open(filename, "w") as f:
        json.dump(data.to_json(), f)
    print("Done.")


EXPORTERS = {
    "csv": export_to_csv,
    "json": export_to_json,
}


def get_exporter(format: str) -> ExportFn:
    """Factory function to get the appropriate exporter function based on format string."""
    if format in EXPORTERS:
        return EXPORTERS[format]
    else:
        raise ValueError(f"Unsupported export format: {format}")


def main() -> None:
    report = Report(
        title="Quarterly Earnings",
        content="Here are the earnings for the last quarter...",
    )

    export_fn = get_exporter("json")
    export_fn("report.json", report)

    budget = Budget(
        title="Annual Budget",
        amount=1000000.00,
    )

    export_fn = get_exporter("json")
    export_fn("budget.json", budget)


if __name__ == "__main__":
    main()
```
• Класс протокола помогает отделить функции от реальных объектов.

# Анти-паттерн 9: Общий Импорт Модулей

• Импорт всего с помощью подстановочного знака.
```python
from dataclasses import *
```
• Проблемы с импортом всего из модуля data classes.
• Рекомендация использовать эксплицитный импорт для ясности.
```python
from dataclasses import dataclass
```
# Анти-паттерн 10: Не Использование Встроенных Инструментов И Библиотек

• Не изобретать велосипед, использовать встроенные инструменты Python.
• Примеры использования os и json.
• Рекомендация ознакомиться с библиотеками перед созданием собственных решений.

24:29 Заключение и призыв к обсуждению

• Подсказки по типу как полезный инструмент.
• Призыв к обсуждению анти-паттернов в комментариях.
• Обещание показать полезность подсказок по типу в следующем видео.

[[ППУ внедрение памятки анти-паттернов Python]]