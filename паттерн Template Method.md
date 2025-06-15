---
tags:
  - АрхитектураПО
---
# Паттерн Template Method (Шаблонный метод)
**Определение:**
Паттерн определяет "скелет" алгоритма в базовом классе, делегируя реализацию некоторых шагов подклассам. Подклассы могут переопределять отдельные шаги алгоритма, не меняя его структуры.

## Ключевые Компоненты:
1. **Абстрактный класс:** Содержит шаблонный метод (`template_method()`) и объявляет примитивные операции (абстрактные/виртуальные методы).
2. **Конкретные классы:** Реализуют недостающие шаги алгоритма.

---

# Пример На Python
```python
from abc import ABC, abstractmethod

class Game(ABC):
    # Шаблонный метод
    def play(self):
        self.initialize()
        self.start_play()
        self.end_play()

    @abstractmethod
    def initialize(self): pass

    @abstractmethod
    def start_play(self): pass

    def end_play(self):  # Опциональный переопределяемый метод
        print("Игра завершена!")

class Chess(Game):
    def initialize(self):
        print("Расстановка фигур на доске.")

    def start_play(self):
        print("Игроки начинают партию в шахматы.")

class Football(Game):
    def initialize(self):
        print("Футболисты выходят на поле.")

    def start_play(self):
        print("Судья дает свисток. Начало матча!")
    
    def end_play):
        print("Финалльный свисток. Матч окончен!")

# Использование
chess = Chess()
chess.play()
# Вывод:
# Расстановка фигур на доске.
# Игроки начинают партию в шахматы.
# Игра завершена!

football = Football()
football.play()
# Вывод:
# Футболисты выходят на поле.
# Судья дает свисток. Начало матча!
# Финалльный свисток. Матч окончен!
```

---

# Пример На C#
```csharp
using System;

abstract class DataProcessor
{
    // Шаблонный метод
    public void Process()
    {
        ReadData();
        ProcessData();
        SaveData();
    }

    protected abstract void ReadData();
    protected abstract void ProcessData();

    protected virtual void SaveData() // Опционально для переопределения
    {
        Console.WriteLine("Данные сохранены в БД.");
    }
}

class XmlProcessor : DataProcessor
{
    protected override void ReadData()
    {
        Console.WriteLine("Чтение XML-данных...");
    }

    protected override void ProcessData()
    {
        Console.WriteLine("Обработка XML-данных...");
    }
}

class CsvProcessor : DataProcessor
{
    protected override void ReadData()
    {
        Console.WriteLine("Чтение CSV-файла...");
    }

    protected override void ProcessData()
    {
        Console.WriteLine("Конвертация CSV в JSON...");
    }

    protected override void SaveData()
    {
        Console.WriteLine("Экспорт в облачное хранилище.");
    }
}

class Program
{
    static void Main()
    {
        DataProcessor xml = new XmlProcessor();
        xml.Process(); 
        // Вывод:
        // Чтение XML-данных...
        // Обработка XML-данных...
        // Данные сохранены в БД.

        DataProcessor csv = new CsvProcessor();
        csv.Process();
        // Вывод:
        // Чтение CSV-файла...
        // Конвертация CSV в JSON...
        // Экспорт в облачное хранилище.
    }
}
```

---

# Против Каких Антипаттернов Применяется?
1. **Duplicate Code (Дублирование кода):**
   Устраняет повторяющиеся участки кода, вынося общую логику в базовый класс.

2. **Spaghetti Code (Спагетти-код):**
   Четко структурирует алгоритм, делая код читаемым и предсказуемым.

3. **Violation of DRY (Нарушение принципа DRY):**
   Избегает дублирования, централизуя общие шаги в шаблонном методе.

4. **God Object (Божественный объект):**
   Декомпозирует монолитные классы, распределяя ответственность между базовым и дочерними классами.

---

# Когда Использовать?
- Когда несколько классов реализуют схожий алгоритм с незначительными вариациями.
- Когда нужно обеспечить расширяемость отдельных шагов алгоритма без изменения его структуры.
- Для контроля точек расширения алгоритма (через `abstract`/`virtual` методы).

⚠️ **Осторожно!** Не используйте, если количество вариаций шагов слишком велико — это может привести к разрастанию иерархии классов.