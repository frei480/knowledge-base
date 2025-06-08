---
tags:
  - АрхитектураПО
---
### Паттерн Factory Method (Фабричный метод)
**Назначение**: Определяет интерфейс для создания объектов, но позволяет подклассам изменять тип создаваемых экземпляров. Делегирует создание объектов наследникам родительского класса.

**Решает проблемы**:
1. Позволяет системе оставаться независимой от способа создания объектов
2. Обеспечивает гибкость при добавлении новых типов продуктов
3. Инкапсулирует логику создания объектов

**Компоненты**:
1. **Product**: Интерфейс создаваемых объектов
2. **ConcreteProduct**: Конкретные реализации продукта
3. **Creator**: Абстрактный класс с фабричным методом
4. **ConcreteCreator**: Реализация фабричного метода для конкретных продуктов

---

### Пример на Python
```python
from abc import ABC, abstractmethod

# Product
class Transport(ABC):
    @abstractmethod
    def deliver(self): pass

# Concrete Products
class Truck(Transport):
    def deliver(self):
        return "Доставка груза по дороге"

class Ship(Transport):
    def deliver(self):
        return "Доставка груза по морю"

# Creator
class Logistics(ABC):
    @abstractmethod
    def create_transport(self) -> Transport:
        pass
    
    def plan_delivery(self):
        transport = self.create_transport()
        return f"Планирование: {transport.deliver()}"

# Concrete Creators
class RoadLogistics(Logistics):
    def create_transport(self) -> Transport:
        return Truck()

class SeaLogistics(Logistics):
    def create_transport(self) -> Transport:
        return Ship()

# Клиентский код
def client_code(logistics: Logistics):
    print(logistics.plan_delivery())

client_code(RoadLogistics())  # Планирование: Доставка груза по дороге
client_code(SeaLogistics())   # Планирование: Доставка груза по морю
```

---

### Пример на C#
```csharp
using System;

// Product
public interface ITransport
{
    string Deliver();
}

// Concrete Products
public class Truck : ITransport
{
    public string Deliver() => "Доставка груза по дороге";
}

public class Ship : ITransport
{
    public string Deliver() => "Доставка груза по морю";
}

// Creator
public abstract class Logistics
{
    public abstract ITransport CreateTransport();
    
    public string PlanDelivery()
    {
        var transport = CreateTransport();
        return $"Планирование: {transport.Deliver()}";
    }
}

// Concrete Creators
public class RoadLogistics : Logistics
{
    public override ITransport CreateTransport() => new Truck();
}

public class SeaLogistics : Logistics
{
    public override ITransport CreateTransport() => new Ship();
}

// Клиент
class Program
{
    static void Main()
    {
        Logistics roadLogistics = new RoadLogistics();
        Console.WriteLine(roadLogistics.PlanDelivery());  // Планирование: Доставка груза по дороге
        
        Logistics seaLogistics = new SeaLogistics();
        Console.WriteLine(seaLogistics.PlanDelivery());   // Планирование: Доставка груза по морю
    }
}
```

---

### Против каких антипаттернов применяется?
1. **God Object (Божественный объект)**  
   Устраняет монолитные конструкции, где один класс отвечает за создание множества несвязанных объектов.

2. **Hard Coding (Жёсткая привязка)**  
   Заменяет прямой инстанцирование классов (`new ConcreteProduct()`) на гибкий механизм создания объектов.

3. **Violation of OCP (Нарушение принципа открытости/закрытости)**  
   Позволяет добавлять новые типы продуктов без изменения существующего кода клиента.

4. **Tight Coupling (Жёсткая связанность)**  
   Клиентский код зависит от абстракций (Product/Creator), а не от конкретных реализаций.

5. **Duplicate Code (Дублирование кода)**  
   Концентрация логики создания объектов в одном месте, устраняя повторение кода инициализации.

---

### Когда использовать?
- Когда заранее неизвестны конкретные типы объектов
- Когда система должна оставаться расширяемой для новых типов продуктов
- Когда нужно предоставить framework для создания семейств связанных объектов
- Когда требуется разделить код создания объектов и бизнес-логику

### Важные нюансы:
1. Отличие от **[[паттерн simple factory|Simple Factory]]**: Factory Method использует наследование, а Simple Factory - композицию
2. Отличие от **[[паттерн Abstract Factory|Abstract Factory]]**: Factory Method создает один продукт, Abstract Factory - семейства продуктов
3. Часто используется вместе с **[[Dependency Injection основы и применение|Dependency Injection]]**