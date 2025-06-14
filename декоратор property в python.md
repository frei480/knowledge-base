---
tags:
  - python
---

В Python @property — это декоратор, который позволяет создавать свойства в классах. Он позволяет использовать методы класса как атрибуты, что делает код более чистым и удобным для чтения. С помощью @property можно контролировать доступ к атрибутам, добавлять логику при получении или установке значений и обеспечивать инкапсуляцию.

Вот пример использования @property:
```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def radius(self):
        """Получить радиус круга."""
        return self._radius

    @radius.setter
    def radius(self, value):
        """Установить радиус круга."""
        if value < 0:
            raise ValueError("Радиус не может быть отрицательным")
        self._radius = value

    @property
    def area(self):
        """Вычислить площадь круга."""
        import math
        return math.pi * (self._radius ** 2)

# Пример использования
circle = Circle(5)
print(circle.radius)  # Получаем радиус
print(circle.area)    # Получаем площадь

circle.radius = 10    # Устанавливаем новый радиус
print(circle.area)    # Площадь с новым радиусом

# circle.radius = -5  # Это вызовет ValueError

```
В этом примере:
-  Мы создали класс Circle с атрибутом _radius.
-  Используя декоратор @property, мы определили метод radius, который позволяет получать значение радиуса.
-  С помощью декоратора @radius.setter мы определили метод для установки значения радиуса с проверкой на отрицательные значения.
-  Также мы создали свойство area, которое вычисляет площадь круга на основе радиуса.
Таким образом, @property помогает сделать код более интуитивным и безопасным.