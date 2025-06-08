---
tags:
  - АрхитектураПО
---

### SOLID Принципы: Борьба с Антипаттернами (Python и C#)

SOLID — это **5 ключевых принципов объектно-ориентированного проектирования**, которые помогают создавать гибкий, поддерживаемый код и избегать опасных антипаттернов. Разберём каждый принцип с примерами на Python и C#.

---

#### **1. Принцип единственной ответственности (SRP)**  

^eb6306

> *Класс должен иметь только одну причину для изменения.*

**Антипаттерн:** God Object (слишком много ответственностей)  
**Решение:** Разделить функциональность на специализированные классы.

**Python Пример:**
```python
# Нарушение: Класс обрабатывает заказ, логирует и сохраняет
class OrderProcessor:
    def process_order(self, order): 
        # логика обработки
        self._log_order(order)
        self._save_to_db(order)
    
    def _log_order(self, order): ...
    def _save_to_db(self, order): ...

# Исправление:
class OrderService:
    def process(self, order): ...

class OrderLogger:
    def log(self, order): ...

class OrderRepository:
    def save(self, order): ...
```

**C# Пример:**
```csharp
// Нарушение
public class OrderProcessor {
    public void ProcessOrder(Order order) {
        Process(order);
        Log(order);
        Save(order);
    }
    private void Log(Order order) {...}
    private void Save(Order order) {...}
}

// Исправление
public class OrderService {
    public void Process(Order order) {...}
}

public class OrderLogger {
    public void Log(Order order) {...}
}

public class OrderRepository {
    public void Save(Order order) {...}
}
```

---

#### **2. Принцип открытости/закрытости (OCP)**  

^97ae80

> *Программные сущности открыты для расширения, но закрыты для модификации.*

**Антипаттерн:** Жесткая привязка к реализациям  
**Решение:** Использовать абстракции и [[паттерн стратегия|паттерн Стратегия]].

**Python Пример:**
```python
# Нарушение: Добавление нового типа требует изменения класса
class PaymentProcessor:
    def process(self, payment_type):
        if payment_type == "credit_card":
            ...
        elif payment_type == "paypal":
            ...

# Исправление:
from abc import ABC, abstractmethod

class PaymentStrategy(ABC):
    @abstractmethod
    def process(self): ...

class CreditCardStrategy(PaymentStrategy): ...
class PayPalStrategy(PaymentStrategy): ...

class PaymentProcessor:
    def process(self, strategy: PaymentStrategy):
        strategy.process()
```

**C# Пример:**
```csharp
// Нарушение
public class PaymentProcessor {
    public void Process(string paymentType) {
        if (paymentType == "CreditCard") {...}
        else if (paymentType == "PayPal") {...}
    }
}

// Исправление
public interface IPaymentStrategy {
    void Process();
}

public class CreditCardStrategy : IPaymentStrategy {...}
public class PayPalStrategy : IPaymentStrategy {...}

public class PaymentProcessor {
    public void Process(IPaymentStrategy strategy) {
        strategy.Process();
    }
}
```

---

#### **3. Принцип подстановки Барбары Лисков (LSP)**  

^90e45b

> *Объекты должны быть заменяемы экземплярами подклассов без изменения корректности.*

**Антипаттерн:** Нарушение контракта базового класса  
**Решение:** Подклассы не должны ужесточать предусловия или ослаблять постусловия.

**Python Пример:**
```python
# Нарушение: Квадрат меняет поведение родителя
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height

class Square(Rectangle):
    def __init__(self, size):
        super().__init__(size, size)
    
    @property
    def width(self):
        return self._width
    
    @width.setter
    def width(self, value):
        self._width = self._height = value  # Нарушение LSP!

# Исправление: Не наследовать квадрат от прямоугольника
```

**C# Пример:**
```csharp
// Нарушение
public class Rectangle {
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }
}

public class Square : Rectangle {
    public override int Width {
        set { base.Width = value; base.Height = value; }
    }
}

// Исправление: Отдельные классы или интерфейс IShape
```

---

#### **4. Принцип разделения интерфейсов (ISP)**  

^4c6f6c

> *Много специализированных интерфейсов лучше одного универсального.*

**Антипаттерн:** "Толстые" интерфейсы  
**Решение:** Разделить интерфейсы по функциональности.

**Python Пример:**
```python
# Нарушение: Интерфейс заставляет реализовывать ненужные методы
class Worker(ABC):
    @abstractmethod
    def work(self): ...
    @abstractmethod
    def eat(self): ...

class Robot(Worker):  # Робот не должен есть!
    def work(self): ...
    def eat(self): raise NotImplementedError

# Исправление:
class Workable(ABC):
    @abstractmethod
    def work(self): ...

class Eatable(ABC):
    @abstractmethod
    def eat(self): ...

class Human(Workable, Eatable): ...
class Robot(Workable): ...
```

**C# Пример:**
```csharp
// Нарушение
public interface IWorker {
    void Work();
    void Eat();
}

public class Robot : IWorker { // Вынужден реализовывать Eat()
    public void Work() {...}
    public void Eat() => throw new NotSupportedException();
}

// Исправление
public interface IWorkable {
    void Work();
}

public interface IEatable {
    void Eat();
}

public class Human : IWorkable, IEatable {...}
public class Robot : IWorkable {...}
```

---

#### **5. Принцип инверсии зависимостей (DIP)**  

^905e95

> *Зависимости на абстракциях, а не на реализациях.*

**Антипаттерн:** Жесткая связанность (Tight Coupling)  
**Решение:** Внедрение зависимостей через интерфейсы.

**Python Пример:**
```python
# Нарушение: Прямая зависимость от конкретного сервиса
class ReportGenerator:
    def __init__(self):
        self.database = MySQLDatabase()  # Жесткая привязка

    def generate_report(self):
        data = self.database.query_data()
        ...

# Исправление:
class Database(ABC):
    @abstractmethod
    def query_data(self): ...

class MySQLDatabase(Database): ...
class PostgreSQLDatabase(Database): ...

class ReportGenerator:
    def __init__(self, database: Database):  # Зависимость от абстракции
        self.database = database
```

**C# Пример:**
```csharp
// Нарушение
public class ReportGenerator {
    private readonly MySQLDatabase _database;
    public ReportGenerator() {
        _database = new MySQLDatabase();
    }
}

// Исправление
public interface IDatabase {
    List<Data> QueryData();
}

public class ReportGenerator {
    private readonly IDatabase _database;
    public ReportGenerator(IDatabase database) { // DI
        _database = database;
    }
}
```

---

### Как SOLID борется с антипаттернами:
1. **SRP** → Устраняет [[опасные антипаттерны в ООП#^006b99|God Object]]
2. **OCP** → Предотвращает [[опасные антипаттерны в ООП#^5d4656|shotgun surgery]]
3. **LSP** → Решает проблемы хрупких иерархий
4. **ISP** → Борется с "толстыми" интерфейсами
5. **DIP** → Устраняет жесткую связанность

**Главное преимущество:** Код становится:
- Легче тестировать (можно подменять зависимости)
- Проще расширять (через новые реализации интерфейсов)
- Удобнее поддерживать (меньше связанность)
- Безопаснее изменять (меньше риска сломать существующую функциональность)

Пример комплексного применения на C#:
```csharp
// SOLID-совместимый сервис
public class OrderService {
    private readonly IOrderRepository _repository;
    private readonly IPaymentGateway _paymentGateway;
    
    // DI через конструктор
    public OrderService(
        IOrderRepository repository, 
        IPaymentGateway paymentGateway) 
    {
        _repository = repository;
        _paymentGateway = paymentGateway;
    }

    public void ProcessOrder(Order order) {
        if (order.IsValid()) { // SRP: Валидация внутри Order
            _paymentGateway.Process(order); // DIP
            _repository.Save(order); // DIP
        }
    }
}
```