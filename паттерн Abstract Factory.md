---
tags:
  - АрхитектураПО
---
### Паттерн Abstract Factory (Абстрактная фабрика)
**Назначение**: Предоставляет интерфейс для создания семейств связанных или зависимых объектов без указания их конкретных классов. Работает с группами продуктов.

**Решает проблемы**:
1. Создание совместимых семейств объектов (например, UI-элементов одной темы)
2. Изоляция клиентского кода от конкретных классов продуктов
3. Обеспечение согласованности создаваемых объектов

**Компоненты**:
1. **AbstractFactory**: Интерфейс для создания продуктов
2. **ConcreteFactory**: Реализация создания семейства продуктов
3. **AbstractProduct**: Интерфейс для типа продукта
4. **ConcreteProduct**: Конкретные объекты семейства
5. **Client**: Использует фабрику и продукты через абстракции

---

### Пример на Python (Кросс-платформенные UI-элементы)
```python
from abc import ABC, abstractmethod

# Абстрактные продукты
class Button(ABC):
    @abstractmethod
    def render(self): pass

class Checkbox(ABC):
    @abstractmethod
    def paint(self): pass

# Конкретные продукты Windows
class WindowsButton(Button):
    def render(self):
        return "Windows кнопка"

class WindowsCheckbox(Checkbox):
    def paint(self):
        return "Windows чекбокс"

# Конкретные продукты macOS
class MacOSButton(Button):
    def render(self):
        return "macOS кнопка"

class MacOSCheckbox(Checkbox):
    def paint(self):
        return "macOS чекбокс"

# Абстрактная фабрика
class GUIFactory(ABC):
    @abstractmethod
    def create_button(self) -> Button: pass
    
    @abstractmethod
    def create_checkbox(self) -> Checkbox: pass

# Конкретные фабрики
class WindowsFactory(GUIFactory):
    def create_button(self):
        return WindowsButton()
    
    def create_checkbox(self):
        return WindowsCheckbox()

class MacOSFactory(GUIFactory):
    def create_button(self):
        return MacOSButton()
    
    def create_checkbox(self):
        return MacOSCheckbox()

# Клиентский код
def create_ui(factory: GUIFactory):
    button = factory.create_button()
    checkbox = factory.create_checkbox()
    print(f"{button.render()} + {checkbox.paint()}")

create_ui(WindowsFactory())  # Windows кнопка + Windows чекбокс
create_ui(MacOSFactory())    # macOS кнопка + macOS чекбокс
```

---

### Пример на C# (Мебельные стили)
```csharp
using System;

// Абстрактные продукты
public interface IChair
{
    string Style { get; }
}

public interface ISofa
{
    string Design();
}

// Конкретные продукты Victorian
public class VictorianChair : IChair
{
    public string Style => "Викторианский стул";
}

public class VictorianSofa : ISofa
{
    public string Design() => "Диван с резными ножками";
}

// Конкретные продукты Modern
public class ModernChair : IChair
{
    public string Style => "Стул в стиле модерн";
}

public class ModernSofa : ISofa
{
    public string Design() => "Минималистичный диван";
}

// Абстрактная фабрика
public interface IFurnitureFactory
{
    IChair CreateChair();
    ISofa CreateSofa();
}

// Конкретные фабрики
public class VictorianFurnitureFactory : IFurnitureFactory
{
    public IChair CreateChair() => new VictorianChair();
    public ISofa CreateSofa() => new VictorianSofa();
}

public class ModernFurnitureFactory : IFurnitureFactory
{
    public IChair CreateChair() => new ModernChair();
    public ISofa CreateSofa() => new ModernSofa();
}

// Клиент
class Program
{
    static void CreateRoom(IFurnitureFactory factory)
    {
        var chair = factory.CreateChair();
        var sofa = factory.CreateSofa();
        Console.WriteLine($"{chair.Style} и {sofa.Design()}");
    }

    static void Main()
    {
        CreateRoom(new VictorianFurnitureFactory());
        // Викторианский стул и Диван с резными ножками
        
        CreateRoom(new ModernFurnitureFactory());
        // Стул в стиле модерн и Минималистичный диван
    }
}
```

---

### Против каких антипаттернов применяется?
1. **God Object (Божественный объект)**  
   Распределяет создание объектов по специализированным фабрикам вместо одного "монолитного" создателя.

2. **Hard Coding (Жёсткая привязка)**  
   Устраняет прямые зависимосити вида `new VictorianChair()` в клиентском коде.

3. **Inconsistent State (Несогласованное состояние)**  
   Гарантирует создание совместимых объектов (например, не позволяет смешать Victorian и Modern мебель).

4. **Violation of OCP (Нарушение OCP)**  
   Позволяет добавлять новые семейства продуктов без изменения существующего кода.

5. **Spaghetti Code (Спагетти-код)**  
   Инкапсулирует сложную логику создания связанных объектов в одном месте.

---

### Когда использовать?
- Система должна быть независимой от процесса создания объектов
- Требуется создавать семейства связанных объектов
- Продукты должны использоваться вместе
- Нужно предоставить библиотеку продуктов, раскрывая только интерфейсы

### Отличия от [[паттерн Factory Method|Factory Method]]:
| **Критерий**       | **Factory Method**               | **Abstract Factory**               |
|---------------------|----------------------------------|------------------------------------|
| Уровень абстракции | Создание одного продукта         | Создание семейств продуктов        |
| Реализация         | Наследование                     | Композиция + наследование          |
| Масштаб            | Уровень класса                   | Уровень системы                    |
| Изменение продуктов| Путем подклассов Creator         | Путем замены фабрики целиком       |

### Недостатки:
- Сложность добавления новых типов продуктов (требует изменения всех фабрик)
- Усложнение кода из-за большого количества классов