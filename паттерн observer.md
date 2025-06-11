---
tags:
  - АрхитектураПО
---

# Паттерн Observer (Наблюдатель)
**Назначение:**
Определяет зависимость "один-ко-многим" между объектами так, что при изменении состояния одного объекта (субъекта) все зависящие от него объекты (наблюдатели) автоматически уведомляются и обновляются.

## Ключевые Аспекты:
1. **Субъект (Subject):** Хранит состояние и управляет подписками наблюдателей
2. **Наблюдатель (Observer):** Интерфейс для объектов, получающих обновления
3. **Уведомление:** Автоматическая рассылка при изменении состояния
4. **Слабая связанность:** Субъект не знает деталей реализации наблюдателей

---

# Пример На Python
**Сценарий:** Система оповещения о погоде (метеостанция уведомляет устройства)

```python
from abc import ABC, abstractmethod

# Интерфейс наблюдателя
class Observer(ABC):
    @abstractmethod
    def update(self, temperature, humidity):
        pass

# Интерфейс субъекта
class Subject(ABC):
    @abstractmethod
    def register_observer(self, observer):
        pass
    
    @abstractmethod
    def remove_observer(self, observer):
        pass
    
    @abstractmethod
    def notify_observers(self):
        pass

# Конкретный субъект
class WeatherStation(Subject):
    def __init__(self):
        self.observers = []
        self.temperature = 0
        self.humidity = 0
    
    def register_observer(self, observer):
        self.observers.append(observer)
    
    def remove_observer(self, observer):
        self.observers.remove(observer)
    
    def notify_observers(self):
        for observer in self.observers:
            observer.update(self.temperature, self.humidity)
    
    def set_measurements(self, temperature, humidity):
        self.temperature = temperature
        self.humidity = humidity
        self.notify_observers()

# Конкретные наблюдатели
class PhoneDisplay(Observer):
    def update(self, temperature, humidity):
        print(f"Phone: Температура {temperature}°C, Влажность {humidity}%")

class TVDisplay(Observer):
    def update(self, temperature, humidity):
        print(f"TV: Погода обновлена! {temperature}°C, {humidity}% влажности")

# Клиентский код
station = WeatherStation()
phone = PhoneDisplay()
tv = TVDisplay()

station.register_observer(phone)
station.register_observer(tv)

station.set_measurements(25, 60)
# Вывод:
# Phone: Температура 25°C, Влажность 60%
# TV: Погода обновлена! 25°C, 60% влажности

station.remove_observer(tv)
station.set_measurements(23, 65)
# Вывод:
# Phone: Температура 23°C, Влажность 65%
```

---

# Пример На C#
**Сценарий:** Система уведомлений заказа (склад, логистика, клиент)

```csharp
using System;
using System.Collections.Generic;

// Интерфейс наблюдателя
interface IOrderObserver {
    void Update(string orderId, string status);
}

// Интерфейс субъекта
interface IOrderSubject {
    void RegisterObserver(IOrderObserver observer);
    void RemoveObserver(IOrderObserver observer);
    void NotifyObservers();
}

// Конкретный субъект
class Order : IOrderSubject {
    private List<IOrderObserver> _observers = new List<IOrderObserver>();
    public string OrderId { get; }
    private string _status;
    
    public Order(string id) => OrderId = id;
    
    public void RegisterObserver(IOrderObserver observer) => _observers.Add(observer);
    public void RemoveObserver(IOrderObserver observer) => _observers.Remove(observer);
    
    public void NotifyObservers() {
        foreach (var observer in _observers) {
            observer.Update(OrderId, _status);
        }
    }
    
    public void UpdateStatus(string status) {
        _status = status;
        NotifyObservers();
    }
}

// Конкретные наблюдатели
class Warehouse : IOrderObserver {
    public void Update(string orderId, string status) {
        Console.WriteLine($"Склад: Заказ {orderId} - {status}. Готовим товар к отправке");
    }
}

class Customer : IOrderObserver {
    public void Update(string orderId, string status) {
        Console.WriteLine($"Клиент: Ваш заказ {orderId} обновлен: {status}");
    }
}

// Клиентский код
class Program {
    static void Main() {
        var order = new Order("ORD-12345");
        
        var warehouse = new Warehouse();
        var customer = new Customer();
        
        order.RegisterObserver(warehouse);
        order.RegisterObserver(customer);
        
        order.UpdateStatus("Оплачен");
        order.UpdateStatus("Отправлен");
        
        order.RemoveObserver(warehouse);
        order.UpdateStatus("Доставлен");
    }
}
```

**Вывод:**
```
Склад: Заказ ORD-12345 - Оплачен. Готовим товар к отправке
Клиент: Ваш заказ ORD-12345 обновлен: Оплачен

Склад: Заказ ORD-12345 - Отправлен. Готовим товар к отправке
Клиент: Ваш заказ ORD-12345 обновлен: Отправлен

Клиент: Ваш заказ ORD-12345 обновлен: Доставлен
```

---

# Против Каких Антипаттернов Борется Observer?
1. **[[опасные антипаттерны в ООП#^285f2b|Spaghetti Code (Спагетти-код)]]**
   **Проблема:** Прямые вызовы между объектами создают запутанные зависимости.
   **Решение:** Observer разделяет объекты через механизм подписки/уведомления.

2. **[[опасные антипаттерны в ООП#^19f14b|Tight Coupling (Жёсткая связанность)]]**
   **Проблема:** Объекты напрямую зависят от реализации друг друга.
   **Решение:** Субъект и наблюдатели связаны через абстракции.

3. **Polling (Активный опрос)**
   **Проблема:** Постоянная проверка состояния объекта тратит ресурсы.
   **Решение:** Push-модель - уведомления только при реальных изменениях.

4. **Violation of Open/Closed Principle**
   **Проблема:** Добавление новых получателей требует изменения кода отправителя.
   **Решение:** Новые наблюдатели добавляются без модификации субъекта.

5. **Chain of Responsibility Overuse**
   **Проблема:** Неоправданное использование цепочек вызовов для уведомлений.
   **Решение:** Централизованный механизм подписки/рассылки.

---

# Когда Использовать Observer?
1. При необходимости уведомления множества объектов об изменении состояния
2. В системах событийно-ориентированной архитектуры (GUI, IoT)
3. Когда отправитель не должен знать о получателях
4. Для реализации механизма publish-subscribe
5. В системах, где состояние должно синхронизироваться между компонентами

> **Отличие от [[паттерн mediator|Mediator]]:**
> - Mediator централизует сложные взаимодействия между компонентами
> - Observer фокусируется на односторонней рассылке уведомлений от субъекта к наблюдателям