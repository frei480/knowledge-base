---
tags:
  - АрхитектураПО
  - python
---
**Подробный разбор принципов GRASP с примерами на Python**
_Крэг Ларман, 9 принципов для грамотного ООП-дизайна_

---
**Связь принципов GRASP с шаблонами проектирования**
Рассмотрим также, как принципы GRASP соотносятся с классическими шаблонами проектирования из книги “Банды четырёх” (GoF).

**Связь принципов GRASP с [[SOLID принципы|SOLID]]**
Оба набора принципов направлены на создание гибкого и поддерживаемого кода, но фокусируются на разных аспектах. GRASP отвечает на вопрос _«Кому назначить ответственность?»_, а SOLID — _«Как проектировать классы и зависимости?»_. Рассмотрим их взаимосвязь.
# **1. Information Expert (Информационный эксперт)**

**Суть**: Ответственность должна быть назначена классу, который обладает максимумом необходимой информации для её выполнения.

**Пример**:
```python
class ShoppingCart:
    def __init__(self):
        self.items = []  # Корзина хранит товары

    # Корзина знает свои товары → логично, что она считает итог
    def calculate_total(self):
        return sum(item.price * item.quantity for item in self.items)

class Item:
    def __init__(self, name, price, quantity):
        self.name = name
        self.price = price
        self.quantity = quantity

# Использование
cart = ShoppingCart()
cart.items.append(Item("Book", 20, 2))
cart.items.append(Item("Pen", 5, 3))
print(cart.calculate_total())  # 20*2 + 5*3 = 55
```
---
**Связь с GoF**:
- **[[паттерн composite|Composite (Компоновщик)]]**: Если бы `ShoppingCart` содержал вложенные корзины или группы товаров, использовалась бы древовидная структура.
- **Chain of Responsibility (Цепочка обязанностей)**: Если бы расчет суммы требовал поэтапной обработки (скидки, налоги), цепочка обработчиков могла бы делегировать задачи.

**Связь с SOLID: [[SOLID принципы#**1. Принцип Единственной Ответственности (SRP)**|Принцип Единственной Ответственности (SRP)]]:**
Класс должен иметь только одну причину для изменения.
    **Пример**:

```python
class Order:
    def __init__(self, items):
        self.items = items  # Order владеет данными

    # Information Expert: Order вычисляет сумму
    def total(self):
        return sum(item.price * item.quantity for item in self.items)
```

**Связь**: `Order` отвечает только за расчет суммы (SRP), так как владеет данными (Information Expert).
# **2. Creator (Создатель)**

**Суть**: Класс `A` должен создавать объекты класса `B`, если:
- `A` содержит или агрегирует `B`;
- `A` использует `B` напрямую;
- `A` владеет данными для инициализации `B`.
    
**Пример**:
```python
class Library:
    def __init__(self):
        self.books = []

    # Library создаёт Book, так как управляет коллекцией книг
    def add_book(self, title, author):
        self.books.append(Book(title, author))

class Book:
    def __init__(self, title, author):
        self.title = title
        self.author = author

# Использование
lib = Library()
lib.add_book("Clean Code", "Robert Martin")
print(lib.books[0].title)  # Clean Code
```
---
**Связь с GoF**:
- **[[паттерн Factory Method|Factory Method (Фабричный метод)]]**: Метод `add_book` действует как фабрика для создания `Book`.
- **[[паттерн singleton|Singleton (Одиночка)]]**: Если бы `Library` была единственным источником создания книг в системе.

**Связь с SOLID**: Зависимости должны строиться на абстракциях. [[SOLID принципы#**5. Принцип Инверсии Зависимостей (DIP)**|Принцип Инверсии Зависимостей]]
**Пример**:
```python
class User:
    def __init__(self, db):  # Внедрение зависимости через интерфейс
        self.db = db

    def save(self):
        self.db.save(self)
```

**Связь**: `User` не создает `db` напрямую (Creator), а получает его через интерфейс (DIP).
# **3. Low Coupling (Слабое зацепление)**

**Суть**: Минимизируйте зависимости между классами. Чем меньше связей — тем проще изменять систему.

**Пример**:
```python
# Плохо: сильная связь между User и Database
class User:
    def __init__(self):
        self.db = Database()  # Прямая зависимость

    def save(self):
        self.db.save_user(self)

# Хорошо: используем Dependency Injection
class User:
    def save(self, db):
        db.save_user(self)  # Зависимость через интерфейс

class Database:
    def save_user(self, user):
        print(f"Saving {user} to DB")
```

---
**Связь с GoF**:
- **[[паттерн стратегия|Strategy (Стратегия)]]**: `Database` можно рассматривать как стратегию для сохранения данных.
- **[[паттерн Adapter|Adapter (Адаптер)]]**: Если бы `User` работал с разными типами БД через унифицированный интерфейс.

**Связь с SOLID**: [[SOLID принципы#**5. Принцип Инверсии Зависимостей (DIP)**|Зависеть от абстракций]] и [[SOLID принципы#**4. Принцип Разделения Интерфейсов (ISP)**|разделять интерфейсы]].
**Пример**:
```python
class PaymentGateway(ABC):  # Абстракция (DIP)
    @abstractmethod
    def pay(self, amount):
        pass

class StripeGateway(PaymentGateway):  # Узкий интерфейс (ISP)
    def pay(self, amount):
        print(f"Paying ${amount} via Stripe")
```

**Связь**: Слабая связь (Low Coupling) достигается через абстракции (DIP) и разделение интерфейсов (ISP).
# **4. High Cohesion (Высокая связность)**

**Суть**: Класс должен решать одну четкую задачу. Все его методы связаны с этой задачей.
**Пример**:
```python
# Плохо: класс делает всё подряд
class ReportManager:
    def generate_report(self): ...
    def send_email(self): ...     # Не относится к отчетам
    def format_data(self): ...

# Хорошо: разделяем ответственности
class ReportGenerator:
    def generate(self): ...

class EmailService:
    def send(self): ...
```
---
**Связь с GoF**:
- **[[паттерн Facade|Facade (Фасад)]]**: `ReportGenerator` может предоставлять упрощенный интерфейс для сложной логики генерации отчетов.
- **[[паттерн command|Command (Команда)]]**: Каждое действие (генерация отчета, отправка email) могло бы быть инкапсулировано в отдельные объекты-команды.

**Связь с SOLID**: [[SOLID принципы#**1. Принцип Единственной Ответственности (SRP)**|Принцип Единственной Ответственности (SRP)]].
**Пример**:
```python
class ReportGenerator:  # Только генерация отчетов
    def generate(self, data):
        ...

class EmailService:  # Только отправка писем
    def send(self, recipient, message):
        ...
```

**Связь**: Высокая связность (High Cohesion) = Соблюдение SRP.
# **5. Controller (Контроллер)**

**Суть**: Используйте отдельный класс для обработки системных событий (например, HTTP-запросов).

**Пример**:
```python
class PaymentController:
    def process_payment(self, request):
        # Валидация, логика оплаты, запись в БД
        if self.validate(request):
            payment = Payment(request.amount)
            payment.execute()
            return "Success"
        return "Error"

class Payment:
    def __init__(self, amount):
        self.amount = amount

    def execute(self):
        print(f"Processing payment: ${self.amount}")
```
---
**Связь с GoF**:
- **[[паттерн command|Command (Команда)]]**: Запрос на оплату можно инкапсулировать в объект-команду.
- **[[паттерн mediator|Mediator (Посредник)]]**: Контроллер может координировать взаимодействие между `Payment`, `БД` и другими сервисами.

**Связь с SOLID**: Классы [[SOLID принципы#**2. Принцип открытости/закрытости (OCP)**|открыты для расширения, но закрыты для изменений]].
**Пример**:
```python
class PaymentController:
    def process(self, payment: PaymentGateway):  # Принимает абстракцию (OCP)
        payment.pay(100)
```

**Связь**: Контроллер можно расширить новыми типами платежей, не меняя его код (OCP).
# **6. Polymorphism (Полиморфизм)**
**Суть**: Используйте интерфейсы и переопределение методов для обработки разных типов объектов.
**Пример**:
```python
class Shape:
    def area(self):
        raise NotImplementedError

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return 3.14 * self.radius ** 2

class Square(Shape):
    def __init__(self, side):
        self.side = side

    def area(self):
        return self.side ** 2

# Клиентский код работает с любыми Shape
shapes = [Circle(5), Square(4)]
print(sum(shape.area() for shape in shapes))  # 78.5 + 16 = 94.5
```
---
**Связь с GoF**:
- **[[паттерн стратегия|Strategy (Стратегия)]]**: Расчет площади — это стратегия, специфичная для каждой фигуры.
- **Template Method (Шаблонный метод)**: Если бы `Shape` имел общий алгоритм с шагами, переопределяемыми в подклассах.

**Связь с SOLID**: [[SOLID принципы#**3. Принцип Подстановки Барбары Лисков (LSP)**|Подклассы должны заменять базовые классы]].
**Пример**:
```python
class Bird:
    def fly(self):
        ...

class Sparrow(Bird):  # LSP: корректная замена
    def fly(self):
        print("Flying")

class Ostrich(Bird):  # Нарушает LSP!
    def fly(self):
        raise NotImplementedError("Ostriches can't fly")
```

**Связь**: Полиморфизм (GRASP) требует соблюдения LSP.
# **7. Pure Fabrication (Чистая выдумка)**
**Суть**: Создавайте классы, не имеющие аналогов в предметной области, но упрощающие систему.
**Пример**:
```python
# Сервис для работы с API — не часть предметной области, но полезен
class WeatherAPIClient:
    def get_forecast(self, city):
        # Логика запроса к внешнему API
        return {"city": city, "temp": 25}

class WeatherService:
    def __init__(self):
        self.api = WeatherAPIClient()

    def show_forecast(self, city):
        data = self.api.get_forecast(city)
        print(f"Temperature in {city}: {data['temp']}°C")
```
---
**Связь с GoF**:
- **[[паттерн Adapter|Adapter (Адаптер)]]**: Если бы `WeatherAPIClient` адаптировал данные внешнего API к внутреннему формату.
- **[[паттерн proxy|Proxy (Заместитель)]]**: Сервис мог бы кэшировать запросы или контролировать доступ к API.

**Связь с SOLID**: [[SOLID принципы#**4. Принцип Разделения Интерфейсов (ISP)**|Интерфейсы должны быть узкоспециализированными]].
**Пример**:
```python
class JSONParser:  # «Выдуманный» сервис
    def parse(self, data):
        ...

class XMLParser:
    def parse(self, data):
        ...
```

**Связь**: Узкие интерфейсы парсеров (ISP) делают их проще в поддержке (Pure Fabrication).
# **8. Indirection (Посредник)**
**Суть**: Вводите промежуточные объекты, чтобы избежать прямых связей между компонентами.
**Пример**:
```python
# Посредник для обработки событий
class EventBus:
    def __init__(self):
        self.subscribers = []

    def subscribe(self, handler):
        self.subscribers.append(handler)

    def publish(self, event):
        for handler in self.subscribers:
            handler(event)

# Компоненты не зависят друг от друга
class Logger:
    def log_event(self, event):
        print(f"Log: {event}")

bus = EventBus()
bus.subscribe(Logger().log_event)
bus.publish("User logged in")
```
---
**Связь с GoF**:
- **[[паттерн observer|Observer (Наблюдатель)]]**: `EventBus` реализует механизм подписки/уведомления.
- **[[паттерн mediator|Mediator (Посредник)]]**: Централизует взаимодействие между компонентами.

**Связь с SOLID**: [[SOLID принципы#**5. Принцип Инверсии Зависимостей (DIP)**|Зависимости через абстракции]].
**Пример**:
```python
class EventBus:  # Посредник
    def __init__(self):
        self.subscribers = []

    def subscribe(self, handler):
        self.subscribers.append(handler)

class Logger:
    def log(self, event):
        print(f"Log: {event}")

# Использование
bus = EventBus()
bus.subscribe(Logger().log)  # Зависимость через абстракцию (DIP)
```

**Связь**: Посредник (Indirection) внедряется через интерфейсы (DIP).
# **9. Protected Variations (Защита От изменений)**

**Суть**: Изолируйте компоненты от изменений в других частях системы через абстракции.

**Пример**:
```python
# Абстракция для защиты от смены платежной системы
class PaymentGateway:
    def pay(self, amount):
        raise NotImplementedError

class StripeGateway(PaymentGateway):
    def pay(self, amount):
        print(f"Paying ${amount} via Stripe")

class PayPalGateway(PaymentGateway):
    def pay(self, amount):
        print(f"Paying ${amount} via PayPal")

class Order:
    def __init__(self, gateway: PaymentGateway):
        self.gateway = gateway

    def checkout(self, amount):
        self.gateway.pay(amount)

# Можно менять платежку, не трогая Order
order = Order(StripeGateway())
order.checkout(100)  # Paying $100 via Stripe
```
---
**Связь с GoF**:
- **Bridge (Мост)**: Отделяет абстракцию (`PaymentGateway`) от реализации (`StripeGateway`, `PayPalGateway`).
- **[[паттерн стратегия|Strategy (Стратегия)]]**: Разные платежные системы — это разные стратегии оплаты.

**Связь с SOLID**: [[SOLID принципы#**2. Принцип открытости/закрытости (OCP)**|Система расширяется без модификации существующего кода]].
**Пример**:
```python
class DataExporter:  # Базовый класс
    def export(self, data):
        ...

class CSVExporter(DataExporter):
    def export(self, data):
        print("Exporting to CSV")

class JSONExporter(DataExporter):
    def export(self, data):
        print("Exporting to JSON")
```

**Связь**: Новые экспортеры добавляются без изменения клиентского кода (OCP).
# Советы По Применению GRASP

1. **Не следуйте слепо**: Принципы — это ориентиры, а не строгие правила.
2. **Комбинируйте**: Например, _Low Coupling_ и _High Cohesion_ часто работают вместе.
3. **Анализируйте контекст**: Выбор принципа зависит от конкретной задачи.
    

**Пример комплексного применения**:

```python
# Information Expert (Order знает свои items)
# Creator (Order создаёт OrderItem)
# Low Coupling (Order не зависит от способа оплаты)
class Order:
    def __init__(self):
        self.items = []

    def add_item(self, product, quantity):
        self.items.append(OrderItem(product, quantity))

    def total(self):
        return sum(item.cost() for item in self.items)

class OrderItem:
    def __init__(self, product, quantity):
        self.product = product
        self.quantity = quantity

    def cost(self):
        return self.product.price * self.quantity

class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price
```

GRASP помогает создавать гибкие и понятные системы.
## Как GRASP-принципы Влияют На Cohesion И Coupling

| Принцип              | Влияние на Cohesion и Coupling                                     |
| -------------------- | ------------------------------------------------------------------ |
| Information Expert   | Повышает Cohesion — поведение рядом с данными                      |
| Creator              | Снижает Coupling — создание у ближайшего соседа                    |
| Pure Fabrication     | Повышает Cohesion — выделение искусственного, но логичного объекта |
| Indirection          | Снижает Coupling — прослойка между компонентами                    |
| Controller           | Снижает Coupling UI и бизнес-логика, повышает Cohesion на входе    |
| Polymorphism         | Снижает Coupling, повышает расширяемость и заменяемость            |
| Protected Variations | Локализует Coupling, создает буферы для изменений                  |
# Выводы
## Итоговая Таблица Связей C Шаблонами Проектирования

| **GRASP-принцип**    | **Паттерны GoF**                                                             |
| -------------------- | ---------------------------------------------------------------------------- |
| Information Expert   | [[паттерн composite\|Composite]], Chain of Responsibility                    |
| Creator              | [[паттерн Factory Method\|Factory Method]], [[паттерн singleton\|Singleton]] |
| Low Coupling         | [[паттерн стратегия\|Strategy]], [[паттерн Adapter\|Adapter]]                |
| High Cohesion        | [[паттерн Facade\|Facade]], [[паттерн command\|Command]]                     |
| Controller           | [[паттерн command\|Command]], [[паттерн mediator\|Mediator]]                 |
| Polymorphism         | [[паттерн стратегия\|Strategy]], Template Method                             |
| Pure Fabrication     | [[паттерн Adapter\|Adapter]], [[паттерн proxy\|Proxy]]                       |
| Indirection          | [[паттерн observer\|Observer]], [[паттерн mediator\|Mediator]]               |
| Protected Variations | Bridge, [[паттерн стратегия\|Strategy]]                                      |

---
### Почему Это Важно?
GRASP и GoF-паттерны дополняют друг друга:
- **GRASP** отвечает на вопрос _“Кому назначать ответственность?”_.
- **GoF-паттерны** дают готовые решения _“Как реализовать эту ответственность?”_.

Например, принцип **Protected Variations** подталкивает к использованию **Bridge**, а **Low Coupling** — к внедрению **Strategy**. Это позволяет создавать системы, которые легко расширять и поддерживать.
## Связь С Принципами SOLID

- **GRASP** и **SOLID** взаимно усиливают друг друга. Например:
    - _Information Expert_ и _SRP_ вместе делают классы компактными.
    - _Protected Variations_ и _OCP_ защищают код от изменений.
- Нарушение SOLID часто приводит к нарушению GRASP (и наоборот). Например, сильная связность (Low Coupling) обычно сопровождается нарушением DIP.
- Используйте эти принципы вместе для проектирования систем, которые легко масштабировать и поддерживать.