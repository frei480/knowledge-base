---
tags:
  - АрхитектураПО
---
# Паттерн State (Состояние)
**Назначение:**
Позволяет объекту изменять своё поведение при изменении внутреннего состояния. Создаётся впечатление, что объект изменил свой класс.

**Основные компоненты:**
1. **Контекст (Context):**
   - Хранит ссылку на текущее состояние.
   - Делегирует запросы текущему объекту состояния.
2. **Состояние (State):**
   - Абстрактный класс/интерфейс, определяющий методы для обработки запросов в конкретных состояниях.
3. **Конкретные состояния (Concrete States):**
   - Реализуют поведение, связанное с определённым состоянием контекста.

---

# Пример На Python
```python
from abc import ABC, abstractmethod

# Абстрактное состояние
class State(ABC):
    @abstractmethod
    def handle(self, context):
        pass

# Конкретные состояния
class StateA(State):
    def handle(self, context):
        print("Обработка в StateA. Переход в StateB")
        context.set_state(StateB())

class StateB(State):
    def handle(self, context):
        print("Обработка в StateB. Переход в StateA")
        context.set_state(StateA())

# Контекст
class Context:
    def __init__(self):
        self._state = StateA()  # Начальное состояние

    def set_state(self, state):
        self._state = state

    def request(self):
        self._state.handle(self)

# Клиентский код
context = Context()
context.request()  # StateA -> StateB
context.request()  # StateB -> StateA
```

**Вывод:**
```
Обработка в StateA. Переход в StateB
Обработка в StateB. Переход в StateA
```

---

# Пример На C#
```csharp
using System;

// Абстрактное состояние
public abstract class State
{
    public abstract void Handle(Context context);
}

// Конкретные состояния
public class StateA : State
{
    public override void Handle(Context context)
    {
        Console.WriteLine("Обработка в StateA. Переход в StateB");
        context.SetState(new StateB());
    }
}

public class StateB : State
{
    public override void Handle(Context context)
    {
        Console.WriteLine("Обработка в StateB. Переход в StateA");
        context.SetState(new StateA());
    }
}

// Контекст
public class Context
{
    private State _state;

    public Context()
    {
        _state = new StateA(); // Начальное состояние
    }

    public void SetState(State state)
    {
        _state = state;
    }

    public void Request()
    {
        _state.Handle(this);
    }
}

// Клиентский код
class Program
{
    static void Main()
    {
        var context = new Context();
        context.Request(); // StateA -> StateB
        context.Request(); // StateB -> StateA
    }
}
```

**Вывод:**
```
Обработка в StateA. Переход в StateB
Обработка в StateB. Переход в StateA
```

---

# Против Каких Антипаттернов Применяется State?
1. **Код с дублированием условий (Duplicate Code):**
   Если в разных методах объекта повторяются одинаковые проверки состояния (через `if`/`switch`), State выносит логику каждого состояния в отдельный класс.

2. **Монолитные методы (Long Methods):**
   Большие методы, содержащие ветвления по всем состояниям, разбиваются на компактные классы состояний.

3. **Жёсткая связь (Tight Coupling):**
   Прямые зависимости между контекстом и обработчиками состояний заменяются взаимодействием через абстракцию `State`.

4. **Переключатель типа (Switch on Type):**
   Паттерн устраняет антипаттерн, где поведение определяется через `switch` по типу состояния.

---

# Когда Использовать State?
- Когда поведение объекта зависит от его состояния и должно изменяться во время выполнения.
- Когда в коде много условных операторов, выбирающих поведения в зависимости от текущих полей объекта.
- Когда состояния и переходы между ними должны быть легко расширяемы.

**Плюсы:**
- Изоляция кода для каждого состояния.
- Упрощение контекста.
- Гибкость при добавлении новых состояний.

**Минусы:**
- Избыточность при малом количестве состояний.
- Усложнение структуры кода из-за дополнительных классов.