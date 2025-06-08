### Принцип **Composition over Inheritance** (Композиция вместо Наследования)
Этот принцип рекомендует **предпочитать объединение объектов (композицию)** классическому наследованию для достижения повторного использования кода и гибкости. Основная идея: *"Собирай поведение из отдельных компонентов, а не наследуй его от родителя"*.

---

### **Против каких антипаттернов борется?**
1. **Хрупкий базовый класс (Fragile Base Class)**  
   Изменения в родительском классе ломают наследников.
2. **Глубокие иерархии наследования**  
   Цепочки `A → B → C → D`, где класс `D` зависит от всех вышестоящих.
3. **Нарушение LSP (Liskov Substitution Principle)**  
   Подклассы не могут заменить родителей без изменения поведения.
4. **Жёсткая связанность (Rigid Coupling)**  
   Дочерние классы зависят от реализации родителя.
5. **Взрыв иерархий классов**  
   Множественное наследование или дублирование кода для разных комбинаций признаков.

---

### Примеры реализации

#### **Python**
```python
# Антипаттерн: Наследование для добавления функциональности
class Robot:
    def move(self): 
        print("Moving")

class FlyingRobot(Robot):
    def fly(self): 
        print("Flying")

# Проблема: Что если нужен робот, который ТОЛЬКО летает? 

# Решение через композицию:
class Mover:
    def move(self): 
        print("Moving")

class Flyer:
    def fly(self): 
        print("Flying")

class Robot:
    def __init__(self):
        self.mover = Mover()
        self.flyer = Flyer()
    
    def move(self):
        self.mover.move()
    
    def fly(self):
        self.flyer.fly()

# Гибкое использование:
ground_robot = Robot()
ground_robot.move()  # Работает без летающих компонентов

flying_robot = Robot()
flying_robot.fly()   # Можно использовать только полёт
```

#### **C#**
```csharp
// Антипаттерн: Наследование для разных ролей
public abstract class Employee {
    public void Work() { /* ... */ }
}

public class Manager : Employee { 
    public void ManageTeam() { /* ... */ } 
}

// Проблема: Как добавить навык "кодирование" менеджеру?

// Решение через композицию:
public interface IWorker {
    void Work();
}

public interface ICoder {
    void Code();
}

public class Worker : IWorker {
    public void Work() => Console.WriteLine("Working");
}

public class Coder : ICoder {
    public void Code() => Console.WriteLine("Coding");
}

public class Manager {
    private readonly IWorker _worker;
    private readonly ICoder _coder;

    public Manager(IWorker worker, ICoder coder) {
        _worker = worker;
        _coder = coder;
    }

    public void Manage() {
        _worker.Work();
        Console.WriteLine("Managing team");
    }

    public void WriteCode() => _coder.Code();
}

// Использование:
var manager = new Manager(new Worker(), new Coder());
manager.WriteCode(); // Менеджер может кодировать!
```

---

### **Ключевые преимущества композиции**
| Аспект                | Наследование                                  | Композиция                                     |
|-----------------------|-----------------------------------------------|------------------------------------------------|
| **Гибкость**          | Поведение жёстко связано с иерархией          | Поведение можно менять динамически             |
| **Тестируемость**     | Требует мокирования родительских классов      | Легко внедрять моки через интерфейсы           |
| **Расширяемость**     | Требует создания новых подклассов             | Новые компоненты добавляются без изменений кода |
| **Связанность (Coupling)** | Высокая (наследник зависит от родителя) | Низкая (зависит от абстракций)               |

---

### **Когда использовать наследование?**
1. Отношения **"является" (is-a)** строго соблюдаются: `Dog` → `Animal`.
2. Поведение базового класса **стабильно и не будет меняться**.
3. Нужно **переопределить** методы родителя (полиморфизм).
4. Иерархия **не глубже 2-3 уровней**.

---

### **Итог**
**Composition over Inheritance** — это инструмент против:
- Хрупких базовых классов,
- Громоздких иерархий,
- Нарушений [[SOLID принципы|SOLID]],
- Жёсткой связанности.

Пример: Вместо `class FlyingDog : Dog` лучше создать `class Dog` с компонентом `FlyingBehavior`. Это позволяет создавать собак, которые летают/не летают/плавают, не взрывая иерархию классов.