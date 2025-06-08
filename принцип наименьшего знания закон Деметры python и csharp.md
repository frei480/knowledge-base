### Принцип наименьшего знания (Закон Деметры)
**Основная идея**: Объект должен взаимодействовать только с "ближайшими" объектами:
1. Собственными методами и свойствами
2. Параметрами своих методов
3. Созданными внутри метода объектами
4. Непосредственными компонентами (для композиции)

**Запрещено**: Обращаться к объектам "через точку" (`a.b.c.d`), раскрывая внутреннюю структуру.

---

### Антипаттерны (нарушения)
#### 1. Цепочка вызовов (Train Wreck)
```python
# Python антипаттерн
class Engine:
    def __init__(self):
        self.throttle = Throttle()

class Throttle:
    def set_position(self, pos): ...

class Car:
    def __init__(self):
        self.engine = Engine()

# Нарушение: доступ через 2 уровня
car = Car()
car.engine.throttle.set_position(90)  # car.engine.throttle!
```

```csharp
// C# антипаттерн
public class Engine {
    public Throttle Throttle { get; } = new Throttle();
}

public class Throttle {
    public void SetPosition(int pos) { ... }
}

public class Car {
    public Engine Engine { get; } = new Engine();
}

// Нарушение
var car = new Car();
car.Engine.Throttle.SetPosition(90);  // цепочка из 3 звеньев
```

#### 2. "Посторонний" доступ
```python
# Python антипаттерн
def print_customer_city(order):
    # Прямой доступ к чужой иерархии
    print(order.customer.address.city)  # order -> customer -> address -> city
```

```csharp
// C# антипаттерн
public void PrintCustomerCity(Order order) {
    Console.WriteLine(order.Customer.Address.City);  // 4 уровня вложенности
}
```

---

### Последствия нарушений
1. **Хрупкость кода**: Изменение в `Address` сломает все цепочки.
2. **NullReferenceException**: Если любой элемент в цепочке `null`.
3. **Утечка деталей реализации**: Клиентский код знает о внутренней структуре объектов.
4. **Сложность тестирования**: Требуются глубокие mock-объекты.

---

### Рекомендации и исправления
#### 1. Инкапсуляция поведения в классах
**Python**:
```python
class Car:
    def __init__(self):
        self._engine = Engine()
    
    def set_throttle(self, pos):
        # Делегирование без раскрытия деталей
        self._engine.set_throttle_position(pos)

class Engine:
    def __init__(self):
        self._throttle = Throttle()
    
    def set_throttle_position(self, pos):
        self._throttle.set_position(pos)

# Использование
car = Car()
car.set_throttle(90)  # только одна точка!
```

**C#**:
```csharp
public class Car {
    private readonly Engine _engine = new Engine();
    
    public void SetThrottle(int pos) {
        _engine.SetThrottlePosition(pos);  // одна точка доступа
    }
}

public class Engine {
    private readonly Throttle _throttle = new Throttle();
    
    public void SetThrottlePosition(int pos) {
        _throttle.SetPosition(pos);
    }
}

// Использование
var car = new Car();
car.SetThrottle(90);
```

#### 2. Методы-агрегаторы для данных
**Python**:
```python
class Order:
    def __init__(self, customer):
        self._customer = customer
    
    def get_customer_city(self):
        # Инкапсуляция доступа к данным
        return self._customer.get_city()

class Customer:
    def __init__(self, address):
        self._address = address
    
    def get_city(self):
        return self._address.city  # только одна точка внутри класса
```

**C#**:
```csharp
public class Order {
    private readonly Customer _customer;
    
    public Order(Customer customer) {
        _customer = customer;
    }
    
    public string GetCustomerCity() {
        return _customer.GetCity();
    }
}

public class Customer {
    private readonly Address _address;
    
    public Customer(Address address) {
        _address = address;
    }
    
    public string GetCity() => _address.City;
}
```

#### 3. Паттерн "Представитель" (Delegate)
```python
class CarController:
    def __init__(self, car):
        self._car = car
    
    def accelerate(self):
        self._car.set_throttle(90)  # не знает о Engine/Throttle
```

```csharp
public class CarController {
    private readonly Car _car;
    
    public CarController(Car car) {
        _car = car;
    }
    
    public void Accelerate() {
        _car.SetThrottle(90);  // абстракция над деталями
    }
}
```

---

### Когда можно нарушать закон Деметры
1. **DTO (Data Transfer Objects)**: Простые структуры данных без поведения.
   ```python
   # Python: кортеж/датакласс
   from dataclasses import dataclass

   @dataclass
   class Point:
       x: float
       y: float

   p = Point(1.5, 2.5)
   print(p.x)  # допустимо
   ```
   
   ```csharp
   // C#: record
   public record Point(double X, double Y);
   
   var p = new Point(1.5, 2.5);
   Console.WriteLine(p.X);  // допустимо
   ```

2. **Фреймворки**: LINQ, pandas, где цепочки вызовов - часть DSL:
   ```csharp
   // C# LINQ (исключение из правила)
   var results = data.Where(x => x.IsValid)
                    .OrderBy(x => x.Date)
                    .Select(x => x.Name);
   ```

---

### Итоговые правила
1. **Максимум "одна точка"** в клиентском коде: `object.Method()`.
2. **Не раскрывайте композицию**: Возвращайте интерфейсы, а не реализации.
3. **Создавайте "фасадные" методы**: Для сложных операций над внутренними объектами.
4. **Избегайте "геттеров-туннелей"**: Методы, возвращающие внутренние объекты.

> "Говори только со своими непосредственными друзьями, не болтай с незнакомцами" — классическая формулировка закона.