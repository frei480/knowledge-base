---
tags:
  - АрхитектураПО
---

# Паттерн Mediator (Посредник)
**Назначение:**
Уменьшает связанность между компонентами системы, заставляя их взаимодействовать через централизованный объект-посредник вместо прямых связей. Упрощает изменение взаимодействия между объектами.

## Ключевые Аспекты:
1. **Декаплинг:** Компоненты не ссылаются друг на друга, только на посредника.
2. **Централизация управления:** Посредник координирует взаимодействие.
3. **Упрощение модификации:** Новые правила взаимодействия добавляются в посредник без изменения компонентов.

---

# Пример На Python
**Сценарий:** Система управления воздушным движением (самолёты отправляют сообщения через диспетчера).

```python
# Интерфейс посредника
class AirTrafficControl:
    def register_aircraft(self, aircraft):
        pass
    
    def send_message(self, message, sender):
        pass

# Конкретный посредник
class Tower(AirTrafficControl):
    def __init__(self):
        self.aircrafts = []
    
    def register_aircraft(self, aircraft):
        self.aircrafts.append(aircraft)
        aircraft.set_mediator(self)
    
    def send_message(self, message, sender):
        for aircraft in self.aircrafts:
            if aircraft != sender:
                aircraft.receive(message)

# Компонент системы
class Aircraft:
    def __init__(self, name):
        self.name = name
        self.mediator = None
    
    def set_mediator(self, mediator):
        self.mediator = mediator
    
    def send(self, message):
        print(f"{self.name} отправляет: {message}")
        self.mediator.send_message(message, self)
    
    def receive(self, message):
        print(f"{self.name} получил: {message}")

# Клиентский код
tower = Tower()

aircraft1 = Aircraft("Рейс SU-123")
aircraft2 = Aircraft("Рейс AA-456")
aircraft3 = Aircraft("Рейс LH-789")

tower.register_aircraft(aircraft1)
tower.register_aircraft(aircraft2)
tower.register_aircraft(aircraft3)

aircraft1.send("Запрашиваю разрешение на посадку")
aircraft2.send("Подтверждаю, воздушное пространство свободно")
```

**Вывод:**
```
Рейс SU-123 отправляет: Запрашиваю разрешение на посадку
Рейс AA-456 получил: Запрашиваю разрешение на посадку
Рейс LH-789 получил: Запрашиваю разрешение на посадку

Рейс AA-456 отправляет: Подтверждаю, воздушное пространство свободно
Рейс SU-123 получил: Подтверждаю, воздушное пространство свободно
Рейс LH-789 получил: Подтверждаю, воздушное пространство свободно
```

---

# Пример На C#
**Сценарий:** Чат-комната (пользователи общаются через центральный сервер).

```csharp
// Интерфейс посредника
interface IChatMediator {
    void RegisterUser(User user);
    void SendMessage(string message, User sender);
}

// Конкретный посредник
class ChatRoom : IChatMediator {
    private List<User> _users = new List<User>();
    
    public void RegisterUser(User user) {
        _users.Add(user);
        user.SetMediator(this);
    }
    
    public void SendMessage(string message, User sender) {
        foreach (var user in _users) {
            if (user != sender) {
                user.Receive(message);
            }
        }
    }
}

// Компонент системы
class User {
    private IChatMediator _mediator;
    public string Name { get; }
    
    public User(string name) => Name = name;
    
    public void SetMediator(IChatMediator mediator) => _mediator = mediator;
    
    public void Send(string message) {
        Console.WriteLine($"{Name} отправляет: {message}");
        _mediator.SendMessage(message, this);
    }
    
    public void Receive(string message) {
        Console.WriteLine($"{Name} получил: {message}");
    }
}

// Клиентский код
class Program {
    static void Main() {
        var chat = new ChatRoom();
        
        var user1 = new User("Анна");
        var user2 = new User("Борис");
        var user3 = new User("Виктор");
        
        chat.RegisterUser(user1);
        chat.RegisterUser(user2);
        chat.RegisterUser(user3);
        
        user1.Send("Привет всем!");
        user3.Send("Есть новости по проекту?");
    }
}
```

**Вывод:**
```
Анна отправляет: Привет всем!
Борис получил: Привет всем!
Виктор получил: Привет всем!

Виктор отправляет: Есть новости по проекту?
Анна получил: Есть новости по проекту?
Борис получил: Есть новости по проекту?
```

---

# Против Каких Антипаттернов Борется Mediator?
1. **[[опасные антипаттерны в ООП#^285f2b|Spaghetti Code (Спагетти-код)]]**
   **Проблема:** Прямые связи между множеством объектов создают сеть зависимостей.
   **Решение:** Посредник разрывает прямые связи, централизуя коммуникацию.

2. **[[опасные антипаттерны в ООП#^19f14b|Tight Coupling (Жёсткая связанность)]]**
   **Проблема:** Изменение одного компонента требует модификации всех связанных с ним объектов.
   **Решение:** Компоненты зависят только от посредника, а не друг от друга.

3. **[[опасные антипаттерны в ООП#^006b99|God Object (Божественный объект)]]**
   **Проблема:** Посредник может стать слишком сложным, беря на себя слишком много обязанностей.
   **Решение:** Чёткое разделение - компоненты выполняют свою логику, посредник управляет только коммуникацией.

4. **[[опасные антипаттерны в ООП#^5d4656|Shotgun Surgery (Выстрел дробью)]]**
   **Проблема:** Изменение формата взаимодействия требует правок во многих классах.
   **Решение:** Все изменения в коммуникации локализуются в посреднике.

---

# Когда Использовать Mediator?
1. Когда система состоит из множества компонентов со сложными связями.
2. Когда повторное использование компонентов затруднено из-за их зависимостей.
3. Когда необходимо централизовать управление взаимодействием (логирование, контроль доступа).
4. При реализации распределенных систем или сетевых протоколов.

> **Отличие от [[паттерн Facade|Facade]]:**
> - Facade упрощает доступ к подсистеме, скрывая её сложность.
> - Mediator управляет взаимодействием равноправных компонентов, предотвращая их прямые связи.