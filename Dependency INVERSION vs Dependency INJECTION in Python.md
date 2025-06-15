---
tags:
  - АрхитектураПО
  - python
---

Пересказ видео: https://www.youtube.com/watch?v=2ejbLVkCndI
Dependency INVERSION vs Dependency INJECTION in Python
Автор: Arjan Egges
ИНВЕРСИЯ ЗАВИСИМОСТЕЙ против ВНЕДРЕНИЯ зависимостей в Python

00:00 Введение в зависимости и инверсию зависимостей

- Зависимости в объектно-ориентированном программировании связывают класс с типом объекта.
- Примеры зависимостей: атрибуты, параметры методов, наследование.
- Наследование создает сильную зависимость, поэтому композиция часто предпочтительнее.

00:45 Внедрение зависимостей и инверсия зависимостей

- Внедрение зависимостей перекладывает ответственность за создание объектов на другой класс.
- Это упрощает тестирование кода.
- Принцип инверсии зависимостей является частью [[SOLID принципы|принципов SOLID]] и разделяет конкретные классы с помощью абстракций.
- Без внедрения зависимостей инверсии зависимостей не будет.

02:04 Пример системы заказа и оплаты

- Система включает заказ, авторизатор и платежный процессор.
- Заказ имеет идентификатор и статус платежа.
- Авторизатор генерирует SMS-код и проверяет его.
```python
#before.py
import string
import random

class Order:

    def __init__(self):
        self.id = ''.join(random.choices(string.ascii_lowercase, k=6))
        self.status = "open"

    def set_status(self, status):
        self.status = status

class Authorizer_SMS:

    def __init__(self):
        self.authorized = False
        self.code = None

    def generate_sms_code(self):
        self.code = ''.join(random.choices(string.digits, k=6))

    def authorize(self):
        code = input("Enter SMS code: ")
        self.authorized = code == self.code

    def is_authorized(self) -> bool:
        return self.authorized

class PaymentProcessor:
    
    def pay(self, order):
        authorizer = Authorizer_SMS()
        authorizer.generate_sms_code()
        authorizer.authorize()
        if not authorizer.is_authorized():
            raise Exception("Not authorized")
        print(f"Processing payment for order with id {order.id}")
        order.set_status("paid")
```
03:15 Тестирование системы

- Тестирование заказа проверяет `status` и метод `set_status`.
- Тестирование авторизатора проверяет инициализацию и корректность SMS-кодов.
- Использование фиктивных данных для тестирования авторизации с помощью `patch` используется и как метод и как декоратор.
```python
import unittest
from io import StringIO
from unittest.mock import patch
from before import *

class Order_TestCase(unittest.TestCase):

	def test_init(self):
	order = Order()
	self.assertEqual(order.status, "open")

def test_set_status(self):
	order = Order()
	order.set_status("paid")
	self.assertEqual(order.status, "paid")

class Authorizer_SMS_TestCase(unittest.TestCase):

    def test_init_authorized(self):
        auth = Authorizer_SMS()
        self.assertFalse(auth.is_authorized())
    
    def test_code_decimal(self):
        auth = Authorizer_SMS()
        auth.generate_sms_code()
        self.assertTrue(auth.code.isdecimal())

    def test_authorize_success(self):
        auth = Authorizer_SMS()
        auth.generate_sms_code()
        with patch('builtins.input', return_value=auth.code):
            auth.authorize()
            self.assertTrue(auth.is_authorized())

    @patch('builtins.input', return_value="1234567")
    def test_authorize_fail(self, mocked_input):
        auth = Authorizer_SMS()
        auth.generate_sms_code()
        auth.authorize()
        self.assertFalse(auth.is_authorized())
    
class PaymentProcessor_TestCase(unittest.TestCase):

    def test_payment_success(self):
        # ???
        self.assertTrue(True)

    def test_payment_fail(self):
        # ???
        self.assertTrue(True)


if __name__ == '__main__':
    unittest.main()
```
05:23 Проблемы тестирования платежного процессора

- Метод оплаты `pay` создает авторизатор внутри себя, что затрудняет тестирование.
- Использование макетных объектов для замены зависимостей. Сложно.
- Сложно тестировать функции, которые создают свои собственные объекты, потому что нет возможности их заменить или по крайней мере это не просто.
- На помощь приходит шаблон внедрение зависимостей.

06:27 Внедрение зависимостей в платежный процессор

- Перемещение создания авторизатора `authorizer = Authorizer_SMS()` за пределы платежной системы.
- Передача авторизатора как параметра в инициализаторе платежного процессора.
- Удаление генерации SMS-кода из платежного процессора.
```python
class PaymentProcessor:
    def __init__(self, authorizer: Authorizer_SMS):
	    self.authorizer = authorizer
    def pay(self, order):
        #self.authorizer.generate_sms_code()
        self.authorizer.authorize()
        if not self.authorizer.is_authorized():
            raise Exception("Not authorized")
        print(f"Processing payment for order with id {order.id}")
        order.set_status("paid")
```
08:25 Тестирование платежного процессора

• Добавление теста инициализатора для проверки корректного сохранения авторизатора.
• Тестирование успешной оплаты и проверка статуса заказа.
• Тестирование сбоя платежа с использованием фиктивных данных.
```python
class PaymentProcessor_TestCase(unittest.TestCase):

    def test_init(self):
	    auth = Authorizer_SMS()
	    p = PaymentProcessor(auth)
	    self.assertEqual(p.authorizer, auth)
	    
    def test_payment_success(self):
        auth = Authorizer_SMS()
        auth.generate_sms_code()
        with patch('builtins.input', return_value=auth.code):
	        p = PaymentProcessor(auth)
	        order = Order()
	        p.pay(order)
	        self.assertEqual(order.status, "paid")

    def test_payment_fail(self):
        auth = Authorizer_SMS()
        auth.generate_sms_code()
        with patch('builtins.input', return_value="1234567"):
	        p = PaymentProcessor(auth)
	        order = Order()
	        self.assertRaises(Exception, p.pay, order)
```
10:46 Отчет о покрытии

- Генерация отчета о покрытии в формате HTML.
- Достижение 100-процентного охвата тестов.

11:03 Внедрение зависимостей и его преимущества

- Внедрение зависимостей разделяет обязанности по созданию и использованию объектов.
- Это делает код более тестируемым.
- Обработчик платежей зависит от авторизатора SMS, что ограничивает поддержку других типов авторизации.
 
Благодаря *инверсии зависимостей* мы создаем промежуточный слой, который позволит нам отделить платежный процессор от авторизации.

12:03 Инверсия зависимостей и абстрактный класс

- Введение абстрактного класса Authorizer для разделения обязанностей.
```python
from abc import ABC, abstractmethod


class Authorizer(ABC):
    @abstractmethod
    def authorize(self):
        pass

    @abstractmethod
    def is_authorized(self) -> bool:
        pass
```
• Авторизатор SMS становится подклассом класса Authorizer.
```python
class Authorizer_SMS(Authorizer):
	#внутри без изменений    
```
• Обработчик платежей работает с абстрактным представлением авторизатора.
```python
class PaymentProcessor:
    def __init__(self, authorizer: Authorizer):
        self.authorizer = authorizer
```

12:39 Проблемы с покрытием кода

- Добавление абстрактного класса снижает покрытие кода.
- Абстрактные методы не тестируются, но их можно заменить на строку doc.
```python
class Authorizer(ABC):
    @abstractmethod
    def authorize(self):
        """Autorizes the user"""

    @abstractmethod
    def is_authorized(self) -> bool:
        pass
```
- Создание файла .coveragerc для игнорирования абстрактных методов.
```
.coveragerc
[report]
exclude_lines =
	pragma: no cover
	@abstract
```

14:19 Преимущества инверсии зависимостей

- Инверсия зависимостей позволяет отделить авторизаторы от платежных систем.
- Платежная система знает только о методах авторизации.
- Возможность добавления новых авторизаторов без изменения платежной системы.

14:53 Пример нового авторизатора
• Создание подкласса Authorizer для проверки, является ли пользователь роботом.
```python
class Authorizer_Robot(Authorizer):    
```
• Метод авторизации запрашивает подтверждение у пользователя.
```python
	def authorize(self):
        robot = ""
        while robot != "y" and robot != "n":
            robot = input("Are you a robot (y/n) ?").lower()
        self.authorized = robot == "n"    
```
• Написание тестов для нового авторизатора.
```python
class Authorizer_Robot(Authorizer):

    def __init__(self):
        self.authorized = False

    def authorize(self):
        robot = ""
        while robot != "y" and robot != "n":
            robot = input("Are you a robot (y/n) ?").lower()
        self.authorized = robot == "n"
        
    def is_authorized(self) -> bool:
        return self.authorized
```
15:58 Тестирование нового авторизатора

• Проверка инициализатора и методов авторизации.
• Тестирование успешного и неудачного выполнения авторизации.
• Проверка корректности работы с прописными буквами.
```python
class Authorizer_Robot_TestCase(unittest.TestCase):

    def test_init_authorized(self):
        auth = Authorizer_Robot()
        self.assertFalse(auth.is_authorized())

    def test_authorize_success(self):
        with patch('builtins.input', return_value="n"):
            auth = Authorizer_Robot()
            auth.authorize()
            self.assertTrue(auth.is_authorized())

    def test_authorize_success_uppercase(self):
        with patch('builtins.input', return_value="N"):
            auth = Authorizer_Robot()
            auth.authorize()
            self.assertTrue(auth.is_authorized())

    def test_authorize_fail(self):
        with patch('builtins.input', return_value="y"):
            auth = Authorizer_Robot()
            auth.authorize()
            self.assertFalse(auth.is_authorized())
```
16:51 Заключение

- Внедрение зависимостей отделяет создание объекта от его использования.
- Инверсия зависимостей вводит дополнительный слой для уменьшения зависимости объектов. Дает возможность замены одного объекта подтипом другого.

