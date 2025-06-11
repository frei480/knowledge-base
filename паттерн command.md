---
tags:
  - АрхитектураПО
---
# Паттерн Command (Команда)
**Назначение**: Инкапсулирует запрос как объект, позволяя параметризовать клиенты с различными запросами, ставить запросы в очередь, логировать их и поддерживать отмену операций.

**Решает проблемы**:
1. Разделение инициатора операции (Invoker) и исполнителя (Receiver).
2. Поддержка отмены/повтора операций.
3. Организация очереди команд или транзакций.

**Компоненты**:
1. **Command**: Интерфейс выполнения операции.
2. **ConcreteCommand**: Реализация команды, связывает действие с Receiver.
3. **Invoker**: Инициирует выполнение команды.
4. **Receiver**: Объект, содержащий бизнес-логику.
5. **Client**: Создает команды и связывает их с Receiver.

---

# Пример На Python
```python
from abc import ABC, abstractmethod

# Интерфейс команды
class Command(ABC):
    @abstractmethod
    def execute(self): pass

    @abstractmethod
    def undo(self): pass  # Для отмены

# Receiver (исполнитель)
class Light:
    def on(self):
        print("Свет включен")
    
    def off(self):
        print("Свет выключен")

# ConcreteCommand
class LightOnCommand(Command):
    def __init__(self, light):
        self._light = light
        self._prev_state = None  # Для отмены
    
    def execute(self):
        self._prev_state = "off"
        self._light.on()
    
    def undo(self):
        if self._prev_state == "off":
            self._light.off()

# Invoker (инициатор)
class RemoteControl:
    def __init__(self):
        self._command = None
    
    def set_command(self, command):
        self._command = command
    
    def press_button(self):
        self._command.execute()
    
    def press_undo(self):
        self._command.undo()

# Клиентский код
light = Light()
light_on = LightOnCommand(light)

remote = RemoteControl()
remote.set_command(light_on)

remote.press_button()  # Свет включен
remote.press_undo()    # Свет выключен
```

---

# Пример На C#
```csharp
using System;

// Интерфейс команды
public interface ICommand
{
    void Execute();
    void Undo();
}

// Receiver
public class Light
{
    public void On() => Console.WriteLine("Свет включен");
    public void Off() => Console.WriteLine("Свет выключен");
}

// ConcreteCommand
public class LightOnCommand : ICommand
{
    private Light _light;
    private bool _wasOn; // Состояние для отмены

    public LightOnCommand(Light light) => _light = light;

    public void Execute()
    {
        _wasOn = false; // Предположим, свет был выключен
        _light.On();
    }

    public void Undo()
    {
        if (!_wasOn) 
            _light.Off();
    }
}

// Invoker
public class RemoteControl
{
    private ICommand _command;

    public void SetCommand(ICommand command) => _command = command;

    public void PressButton() => _command.Execute();
    public void PressUndo() => _command.Undo();
}

// Клиент
class Program
{
    static void Main()
    {
        var light = new Light();
        var lightOn = new LightOnCommand(light);
        
        var remote = new RemoteControl();
        remote.SetCommand(lightOn);
        
        remote.PressButton(); // Свет включен
        remote.PressUndo();   // Свет выключен
    }
}
```

---

# Против Каких Антипаттернов Применяется?
1. **[[опасные антипаттерны в ООП#^006b99|God Object (Божественный объект)]]**
   Команда разделяет ответственность: Invoker только инициирует действия, а Receiver содержит логику.

2. **[[опасные антипаттерны в ООП#^19f14b|Tight Coupling (Жёсткая связанность)]]**
   Invoker не зависит от конкретных классов Receiver, взаимодействуя через абстракцию Command.

3. **Long Method (Длинный метод)**
   Каждая команда инкапсулирует отдельное действие, заменяя монолитные методы.

4. **Copy-Paste Programming**
   Повторное использование команд через композицию (макрокоманды) вместо дублирования кода.

5. **Отсутствие отмены операций**
   Паттерн стандартизирует реализацию Undo/Redo через методы в командах.

---

# Когда Использовать?
- Нужна отмена/повтор операций.
- Требуется ставить операции в очередь (например, задачи в пуле потоков).
- Необходимо поддерживать транзакции.
- Требуется параметризация объектов операциями (GUI-кнопки, меню).