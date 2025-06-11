---
tags:
  - АрхитектураПО
---
# Паттерн Simple Factory (Простая фабрика)
**Назначение**: Инкапсулирует логику создания объектов в одном месте, предоставляя клиенту простой интерфейс для создания различных типов объектов без знания их конкретных классов.

**Решает проблемы**:
1. Убирает дублирование кода создания объектов
2. Централизует контроль над процессом инстанцирования
3. Упрощает добавление новых типов продуктов

**Отличие от [[паттерн Factory Method|Factory Method]]**:
- Simple Factory использует один метод для создания всех типов объектов
- Не требует подклассов для изменения поведения (в отличие от Factory Method)
- Часто реализуется через статический метод

---

# Пример На Python
```python
from abc import ABC, abstractmethod

# Абстрактный продукт
class Transport(ABC):
    @abstractmethod
    def deliver(self): pass

# Конкретные продукты
class Truck(Transport):
    def deliver(self):
        return "Доставка грузовиком"

class Ship(Transport):
    def deliver(self):
        return "Доставка кораблем"

class Airplane(Transport):
    def deliver(self):
        return "Доставка самолетом"

# Простая фабрика
class TransportFactory:
    @staticmethod
    def create_transport(transport_type: str) -> Transport:
        if transport_type == "truck":
            return Truck()
        elif transport_type == "ship":
            return Ship()
        elif transport_type == "airplane":
            return Airplane()
        else:
            raise ValueError(f"Неизвестный тип транспорта: {transport_type}")

# Клиентский код
def main():
    transports = [
        TransportFactory.create_transport("truck"),
        TransportFactory.create_transport("ship"),
        TransportFactory.create_transport("airplane")
    ]
    
    for transport in transports:
        print(transport.deliver())

if __name__ == "__main__":
    main()
```

# Пример На C#
```csharp
using System;

// Абстрактный продукт
public interface ITransport
{
    string Deliver();
}

// Конкретные продукты
public class Truck : ITransport
{
    public string Deliver() => "Доставка грузовиком";
}

public class Ship : ITransport
{
    public string Deliver() => "Доставка кораблем";
}

public class Airplane : ITransport
{
    public string Deliver() => "Доставка самолетом";
}

// Простая фабрика
public static class TransportFactory
{
    public static ITransport CreateTransport(string transportType)
    {
        return transportType switch
        {
            "truck" => new Truck(),
            "ship" => new Ship(),
            "airplane" => new Airplane(),
            _ => throw new ArgumentException($"Неизвестный тип транспорта: {transportType}")
        };
    }
}

// Клиент
class Program
{
    static void Main()
    {
        ITransport[] transports = [
            TransportFactory.CreateTransport("truck"),
            TransportFactory.CreateTransport("ship"),
            TransportFactory.CreateTransport("airplane")
        ];

        foreach (var transport in transports)
        {
            Console.WriteLine(transport.Deliver());
        }
    }
}
```

---

# Против Каких Антипаттернов Применяется?
1. **Duplicate Code (Дублирование кода)**
   Устраняет повторяющиеся блоки создания объектов по всей кодовой базе.

2. **Hard Coding (Жёсткая привязка)**
   Заменяет прямые вызовы конструкторов (`new Truck()`) на централизованный фабричный метод.

3. **God Method (Божественный метод)**
   Предотвращает разрастание методов, которые совмещают бизнес-логику с созданием объектов.

4. **Tight Coupling (Жёсткая связанность)**
   Клиенты зависят от абстракции `Transport`, а не от конкретных реализаций.

5. **Shotgun Surgery (Выстрел дробью)**
   Изменения в процессе создания объектов локализованы в одном классе фабрики.

---

# Когда Использовать?
- Когда создание объектов требует сложной логики или условий
- При частом добавлении новых типов продуктов
- Когда в системе много мест, создающих одинаковые объекты
- Для упрощения поддержки и тестирования кода

# Преимущества:
- Простота реализации и понимания
- Улучшенная читаемость кода
- Легкость внедрения изменений
- Упрощение юнит-тестирования

# Недостатки:
- Нарушает OCP при добавлении новых типов (требует изменения фабричного метода)
- Может превратиться в "божественный объект" при чрезмерном использовании
- Не подходит для создания сложных иерархий объектов

# Эволюция Паттерна:
```mermaid
graph LR
    A[Simple Factory] --> B[Factory Method]
    B --> C[Abstract Factory]
```

**Рекомендации**:
- Используйте Simple Factory для простых случаев с 3-5 типами объектов
- Переходите на [[паттерн Factory Method|Factory Method]] при необходимости гибкости через наследование
- Выбирайте [[паттерн Abstract Factory|Abstract Factory]] для создания семейств связанных объектов