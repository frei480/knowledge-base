---
tags:
  - АрхитектураПО
  - python
---
**Подробный разбор принципов GRASP с примерами на Python**
_Крэг Ларман, 9 принципов для грамотного ООП-дизайна_

---

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

# **Советы По Применению GRASP**

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

GRASP помогает создавать гибкие и понятные системы. Начните с малого: применяйте 2–3 принципа, постепенно охватывая остальные.