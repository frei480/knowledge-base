---
tags:
  - python
---
Пересказ видео: https://www.youtube.com/watch?v=zml0rTMJVXg
Автор: Arjan Egges
Видео содержит десять советов для улучшения навыков программирования на Python.

# Первый Совет: Освоить Включения Списков, Словарей, Множеств И Генераторов.
 Списочные включения позволяют создавать списки в одной строке, что делает код более сжатым и читаемым.

Примеры использования списочных включений
- Генерация квадратов четных чисел:
```python
squares = [x**2 for x in range(10) if x % 2 == 0]
print(squares)
```

- Отображение чисел в их кубах:
```python
cubes = {x: x**3 for x in range(5)}
print(cubes)
```

- Уникальная длина слов:
```python
words = ["hello", "world", "python", "hello"]
unique_lengths = {len(word) for word in words}
print(unique_lengths)  # Output: {5, 6}
```

- Генераторы выглядят как списочные включения, но используют скобки. Они вычисляют значения лениво, что делает их эффективными с точки зрения памяти.
```python
squares_gen = (x**2 for x in range(5))
for square in squares_gen:
    print(square)
```

- списочные включения могут быть использованы для многомерной обработки данных.
```python
# Flattening a 2D list:
matrix = [[1, 2], [3, 4], [5, 6]]
flattened = [num for row in matrix for num in row]
print(flattened)
```

# 2 Использование F-строк

• F-строки позволяют форматировать строки с поддержкой чисел, дат и выражений.
```python
name = "Guido"
level = "advanced"
print(f"{name} is an {level} Python programmer!")
```

```shell
Guido is an advanced Python programmer!
```

- Вычисление выражений:
```python
age = 30
print(f"In 10 years, I will be {age + 10} years old.")
```

• Примеры форматирования чисел с определенной точностью и перевода в удобочитаемый формат.
```python
pi = 3.14159
print(f"Pi rounded to two decimals: {pi:.2f}")
```

```shell
Pi rounded to two decimals: 3.14
```

Разделители разрядов:
```python
large_number = 123456789
print(f"Large number with commas: {large_number:,}")
```

Отображение процентов:
```python
success_rate = 0.874
print(f"Success rate: {success_rate:.1%}")
```

• Использование f-строк для форматирования объектов `datetime` и выравнивания текста.
```python
from datetime import datetime

now = datetime.now()
print(f"Current date: {now:%Y-%m-%d}")
print(f"Current time: {now:%H:%M:%S}")
```
```shell
Current date: 2024-12-17
Current time: 11:13:32
```

Выравнивание текста для аккуратного форматирования выводимых данных:
```python
name = "Python"
print(f"{name:<10} is left-aligned")  # Output: Python     is left-aligned
print(f"{name:>10} is right-aligned")  # Output:      Python is right-aligned
print(f"{name:^10} is centered")  # Output:   Python   is centered
```
```shell
Python     is left-aligned
    Python is right-aligned
  Python   is centered
```

Динамическое форматирование ширины:
```python
name = "Arjan"
width = 30
print(f"{name:^{width}}")
```
            Arjan             

• Поддержка синтаксиса отладки для отображения выражений и результатов.
```python
value = 42
print(f"{value=}")
```

```shell
value=42
```

• F-строки не подходят для построения SQL-запросов из-за [[Защита от SQL-инъекций в Python|уязвимости к SQL-инъекциям]].
• Рекомендуется использовать подготовленные выражения: `string.format`, [[шаблонные строки в python|шаблонные строки]].

# Третий Совет: Ознакомиться Со Встроенными Функциями Python, Такими Как Enumerate И Zip.
• Эти функции упрощают код и делают его более понятным.
Вы можете использовать `enumerate` для циклического перебора итерируемого объекта, отслеживая индекс.
```python
items = ["apple", "banana", "cherry"]
for index, item in enumerate(items, start=1):
    print(f"{index}: {item}")
```

Затем есть `zip`, который объединяет два или более итерируемых объекта поэлементно, создавая кортежи:
```python
names = ["Alice", "Bob", "Charlie"]
scores = [85, 90, 95]
for name, score in zip(names, scores):
    print(f"{name}: {score}")
```

```shell
Alice: 85
Bob: 90
Charlie: 95
```

`any` и `all` также весьма полезны:
```python
numbers = [1, 0, 3, 0]
print(any(numbers))  # Output: True (at least one non-zero)
print(all(numbers))  # Output: False (not all are non-zero)
```

```shell
True
False
```

• `Map` преобразует итерацию, применяя функцию к каждому элементу.:
```python
numbers = [1, 2, 3, 4]
squares = map(lambda x: x**2, numbers)
print(list(squares))
```

```shell
[1, 4, 9, 16]
```

• `Filter` выбирает элементы на основе условия, например, числа, делящиеся на 10:
```python
numbers = [10, 15, 20, 25, 30]
divisible_by_10 = filter(lambda x: x % 10 == 0, numbers)
print(list(divisible_by_10))
```

• `Reversed` изменяет порядок элементов, не изменяя их.
```python
items = ["apple", "banana", "cherry"]
for item in reversed(items):
    print(item)
```

```shell
cherry
banana
apple
```

- Min и Max работают с пользовательскими ключами, например, длина слова.
```python
words = ["apple", "banana", "cherry"]
longest_word = max(words, key=len)
print(f"The longest word is {longest_word} with {len(longest_word)} characters.")
```

```shell
The longest word is banana with 6 characters.
```

# Четвертый Совет: Используйте Генераторы Для Эффективной Итерации

• Генераторы используют yield для приостановки функции и генерации значений по запросу.
• Генераторы эффективны для обработки больших наборов данных.
• Пример с квадратными числами, которые вычисляются только при запросе.
```python
# A generator function that yields square numbers
def square_numbers(limit: int):
    for n in range(limit):
        yield n * n

# Use the generator function to compute values
for square in square_numbers(5):
    print(square)
```

```shell
0
1
4
9
16
```
• Генераторы лениво вычисляют значения, что экономит память.
```python
import time

# A generator that computes the square of numbers with a delay
def delayed_squares(limit):
    for n in range(limit):
        print(f"Computing square for: {n}")
        time.sleep(1)  # Simulate a delay in computation
        yield n * n

# Instantiate the generator
gen = delayed_squares(3)

# Step through the generator manually
print("Fetching values on the fly:")
print(next(gen))  # Compute square of 0
print(next(gen))  # Compute square of 1
print(next(gen))  # Compute square of 2
```

```shell
Fetching values on the fly:
Computing square for: 0
0
Computing square for: 1
1
Computing square for: 2
4
```

# Пятый Совет: Контекстные Менеджеры

• Контекстные менеджеры упрощают управление ресурсами, такими как файлы и подключения к базе данных.
• Пример с текстовым файлом, который автоматически закрывается после использования.
```python
with open("example.txt", "r") as file:
    content = file.read()
    print(content)
```
- Контекстные менеджеры могут помочь управлять внешними соединениями, такими как базы данных, гарантируя их правильное закрытие.
```python
import sqlite3

with sqlite3.connect("example.db") as conn:
    cursor = conn.cursor()
    cursor.execute("CREATE TABLE IF NOT EXISTS users (id INTEGER, name TEXT)")
    cursor.execute("INSERT INTO users (id, name) VALUES (1, 'Alice')")
    conn.commit()
    cursor.execute("SELECT * FROM users")
    print(cursor.fetchall())
# The connection is automatically closed at the end of the block.
```

```shell
[(1, 'Alice'), (1, 'Alice'), (1, 'Alice')]
```

• Контекстные менеджеры могут быть созданы с помощью декоратора context manager.
```python
import time
from contextlib import contextmanager

@contextmanager
def timer():
    start = time.time()
    yield
    end = time.time()
    print(f"Elapsed time: {end - start:.2f} seconds")

# Usage:
with timer():
    total = sum(range(10_000_000))
    print(f"Sum computed: {total}")
```

```shell
Sum computed: 49999995000000
Elapsed time: 0.13 seconds
```

Для более сложных сценариев реализуйте методы `__enter__` и `__exit__` непосредственно в классе.
```python
class CustomResource:
    def __enter__(self):
        print("Resource acquired")
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        print("Resource released")

with CustomResource() as resource:
    print("Using the resource")
```

```shell
Resource acquired
Using the resource
Resource released
```
# Шестой Совет: Библиотеки Python

• Python имеет обширную экосистему библиотек, таких как pandas, numpy, httpx, scikit-learn и matplotlib.
• Использование библиотек делает код более эффективным и упрощает работу.
```python
import httpx

response = httpx.get("https://api.github.com")
print(response.status_code)  # Output: 200
print(response.json())  # Output: Parsed JSON response
```
-  Пример использования pandas для манипулирования данными
```python
import pandas as pd

data = {"Name": ["Alice", "Bob"], "Age": [25, 30]}
df = pd.DataFrame(data)
print(df)
# compute the mean of the Age column
print(df["Age"].mean())
```

```shell
    Name  Age
0  Alice   25
1    Bob   30
27.5
```
- и numpy для работы с массивами.
```python
import numpy as np

array = np.array([1, 2, 3])
print(array * 2)
```

```shell
[2 4 6]
```

- `scikit-learn`: Машинное обучение.
```python
from sklearn.linear_model import LinearRegression

model = LinearRegression()
model.fit([[1], [2], [3]], [1, 2, 3])
print(model.predict([[5]]))
```

```shell
[5.]
```

- matplotlib: Построение графиков и диаграмм.
```python
import matplotlib.pyplot as plt

plt.plot([1, 2, 3], [4, 5, 6])
plt.show()
```
# Седьмой Совет: Аннотации Типов

• Аннотации типов помогают другим пользователям и вам лучше понимать код.
• Пример функции, которая принимает список целых чисел и возвращает значение с плавающей запятой.
```python
def average(numbers: list[int]) -> float:
    return sum(numbers) / len(numbers)

print(average([10, 20, 30]))  # Output: 20.0
```

• Аннотации типов делают код более общим и понятным.
```python
from typing import Iterable

def average(numbers: Iterable[int]) -> float:
    return sum(numbers) / len(numbers)

print(average((10, 20, 30)))  # Output: 20.0
```
# Восьмой Совет: Абстракция Кода Для Уменьшения Связности Кода

• Абстракция позволяет отделить функциональность от реализации.
• В Python используются ABC и протоколы для создания чистого и несвязанного кода.
• Пример создания абстрактного базового класса платежного процессора.
```python
from abc import ABC, abstractmethod

class PaymentProcessor(ABC):
    @abstractmethod
    def process_payment(self, amount: float) -> None:
        pass

class PayPalProcessor(PaymentProcessor):
    def process_payment(self, amount: float) -> None:
        print(f"Processing ${amount} payment via PayPal.")

class StripeProcessor(PaymentProcessor):
    def process_payment(self, amount: float) -> None:
        print(f"Processing ${amount} payment via Stripe.")

def process_order(amount: float, processor: PaymentProcessor) -> None:
    processor.process_payment(amount)

paypal = PayPalProcessor()
stripe = StripeProcessor()
process_order(100.0, paypal)
process_order(200.0, stripe)
```
• Абстрактные методы позволяют создавать подклассы для различных реализаций.
• абстрагирование `logger` используя протокол:
```python
from typing import Protocol

class Logger(Protocol):
    def log(self, message: str) -> None:
        pass

class ConsoleLogger:
    def log(self, message: str) -> None:
        print(f"LOG: {message}")

class FileLogger:
    def log(self, message: str) -> None:
        with open("log.txt", "a") as file:
            file.write(f"{message}\n")

def perform_task(logger: Logger) -> None:
    logger.log("Task started.")
    # Simulate a task
    logger.log("Task completed.")

console_logger = ConsoleLogger()
file_logger = FileLogger()
perform_task(console_logger)  # Output: LOG: Task started. LOG: Task completed.
perform_task(file_logger)  # Logs to a file.
```

• Протоколы в Python работают иначе, чем абстрактные классы, и используют аннотации типов .
• Абстракции полезны при продуманном дизайне программного обеспечения.
# Девятый Совет: Тестирование

• Рекомендуется использовать pytest для написания тестов.
• Тесты помогают создавать систему безопасности и масштабируемость кода.

# Десятый Совет: Использование Классов И Функций

• Классы подходят для объединения поведения и данных.
```python
from decimal import Decimal

class ShoppingCart:
    def __init__(self) -> None:
        self.items = []

    def add_item(self, item: str, price: Decimal) -> None:
        self.items.append({"item": item, "price": price})

    def total(self) -> Decimal:
        return sum(item["price"] for item in self.items)

cart = ShoppingCart()
cart.add_item("apple", Decimal("1.5"))
cart.add_item("banana", Decimal("2.0"))
print(f"${cart.total():.2f}")
```
```shell
$3.50
```
• Классы данных `dataclass` упрощают представление структурированных данных.
```python
from dataclasses import dataclass
from decimal import Decimal

@dataclass
class Item:
    name: str
    price: Decimal

item = Item(name="apple", price=Decimal("1.5"))
print(item)
print(f"${item.price:.2f}")
```

```shell
Item(name='apple', price=Decimal('1.5'))
$1.50
```
• Функции полезны для задач без сохранения состояния и могут быть абстрагированы.
```python
def calculate_discounted_price(price: Decimal, discount: Decimal) -> Decimal:
    return price - (price * discount / 100)

discounted_price = calculate_discounted_price(Decimal("100"), Decimal("10"))
print(f"${discounted_price:.2f}")
```

```shell
$90.00
```
• Комбинация классов, данных и функций помогает создавать сложные решения.
```python
from dataclasses import dataclass
from decimal import Decimal

@dataclass
class Item:
    name: str
    price: Decimal

class ShoppingCart:
    def __init__(self) -> None:
        self.items: list[Item] = []

    def add_item(self, item: Item) -> None:
        self.items.append(item)

    def total(self) -> Decimal:
        return sum(item.price for item in self.items)

def calculate_discount(cart: ShoppingCart, discount: Decimal) -> Decimal:
    return cart.total() * (1 - discount / 100)

cart = ShoppingCart()
cart.add_item(Item("apple", Decimal("1.5")))
cart.add_item(Item("banana", Decimal("2.0")))

discounted_price = calculate_discount(cart, Decimal("10"))
print(f"${discounted_price:.2f}")
```

```shell
$3.15
```
# Заключение

• Важно управлять проектом и зависимостями.

[[ППУ внедрение памятки 10 советов для улучшения навыков программирования на Python]]
