---
tags:
  - АрхитектураПО
---

# Паттерн Flyweight (Приспособленец)
**Назначение:**
Экономит память за счёт разделения общих данных между множеством объектов. Позволяет эффективно работать с большим количеством мелких объектов, вынося неизменяемую часть состояния (внутреннее состояние) в разделяемые объекты.

## Ключевые Аспекты:
1. **Разделение состояния:**
   - **Внутреннее состояние:** Неизменяемые данные, общие для многих объектов (разделяются)
   - **Внешнее состояние:** Уникальные данные для каждого объекта (хранятся отдельно)
2. **Фабрика приспособленцев:** Управляет созданием и кэшированием объектов
3. **Экономия памяти:** Устраняет дублирование идентичных данных

---

# Пример На Python
**Сценарий:** Текстовый редактор с оптимизацией хранения символов

```python
from typing import Dict

# Класс приспособленца (хранит внутреннее состояние)
class CharFlyweight:
    def __init__(self, char: str, font: str, size: int):
        self.char = char
        self.font = font
        self.size = size
    
    def render(self, position: int):
        print(f"Символ '{self.char}' ({self.font}-{self.size}pt) на позиции {position}")

# Фабрика приспособленцев
class FlyweightFactory:
    _flyweights: Dict[str, CharFlyweight] = {}
    
    def get_flyweight(self, char: str, font: str, size: int) -> CharFlyweight:
        key = f"{char}_{font}_{size}"
        if key not in self._flyweights:
            self._flyweights[key] = CharFlyweight(char, font, size)
        return self._flyweights[key]
    
    def total_objects(self) -> int:
        return len(self._flyweights)

# Клиентский код (хранит внешнее состояние)
class TextEditor:
    def __init__(self):
        self.factory = FlyweightFactory()
        self.chars = []
    
    def add_char(self, char: str, font: str, size: int, position: int):
        flyweight = self.factory.get_flyweight(char, font, size)
        self.chars.append((flyweight, position))
    
    def render(self):
        for flyweight, position in self.chars:
            flyweight.render(position)

# Использование
editor = TextEditor()
editor.add_char('A', 'Arial', 12, 0)
editor.add_char('B', 'Times New Roman', 14, 1)
editor.add_char('A', 'Arial', 12, 2)  # Использует существующий объект
editor.add_char('C', 'Arial', 12, 3)

editor.render()
print(f"Всего объектов: {editor.factory.total_objects()}")  # Всего объектов: 3
```

**Вывод:**
```
Символ 'A' (Arial-12pt) на позиции 0
Символ 'B' (Times New Roman-14pt) на позиции 1
Символ 'A' (Arial-12pt) на позиции 2
Символ 'C' (Arial-12pt) на позиции 3
Всего объектов: 3
```

---

# Пример На C#
**Сценарий:** Игровой движок с оптимизацией хранения деревьев

```csharp
using System;
using System.Collections.Generic;

// Внутреннее состояние (разделяемые данные)
class TreeType {
    public string Name { get; }
    public string Color { get; }
    public string Texture { get; }
    
    public TreeType(string name, string color, string texture) {
        Name = name;
        Color = color;
        Texture = texture;
    }
    
    public void Draw(int x, int y) {
        Console.WriteLine($"Дерево {Name} ({Color}) на позиции ({x},{y})");
    }
}

// Фабрика приспособленцев
class TreeFactory {
    private static Dictionary<string, TreeType> _types = new();
    
    public static TreeType GetTreeType(string name, string color, string texture) {
        string key = $"{name}_{color}_{texture}";
        if (!_types.ContainsKey(key)) {
            _types[key] = new TreeType(name, color, texture);
        }
        return _types[key];
    }
    
    public static int TotalTypes() => _types.Count;
}

// Клиентский класс (хранит внешнее состояние)
class Tree {
    private TreeType _type;
    private int _x;
    private int _y;
    
    public Tree(int x, int y, TreeType type) {
        _x = x;
        _y = y;
        _type = type;
    }
    
    public void Draw() => _type.Draw(_x, _y);
}

// Клиентский код
class Forest {
    private List<Tree> _trees = new();
    
    public void PlantTree(int x, int y, string name, string color, string texture) {
        var type = TreeFactory.GetTreeType(name, color, texture);
        _trees.Add(new Tree(x, y, type));
    }
    
    public void DrawForest() {
        foreach (var tree in _trees) {
            tree.Draw();
        }
    }
}

class Program {
    static void Main() {
        var forest = new Forest();
        forest.PlantTree(10, 20, "Дуб", "Темно-зеленый", "Дубовая_текстура");
        forest.PlantTree(30, 40, "Сосна", "Зеленый", "Сосновая_текстура");
        forest.PlantTree(50, 60, "Дуб", "Темно-зеленый", "Дубовая_текстура"); // Существующий тип
        forest.PlantTree(70, 80, "Береза", "Белый", "Березовая_текстура");
        
        forest.DrawForest();
        Console.WriteLine($"Всего типов деревьев: {TreeFactory.TotalTypes()}");
    }
}
```

**Вывод:**
```
Дерево Дуб (Темно-зеленый) на позиции (10,20)
Дерево Сосна (Зеленый) на позиции (30,40)
Дерево Дуб (Темно-зеленый) на позиции (50,60)
Дерево Береза (Белый) на позиции (70,80)
Всего типов деревьев: 3
```

---

# Против Каких Антипаттернов Борется Flyweight?
1. **Memory Hog (Пожиратель памяти):**
   **Проблема:** Приложение создает чрезмерное количество объектов, что приводит к перерасходу памяти.
   **Решение:** Разделение общего состояния между объектами.

2. **Object Orgy (Объектная оргия):**
   **Проблема:** Бесконтрольное создание объектов с дублирующимися данными.
   **Решение:** Централизованное управление общими состояниями через фабрику.

3. **Premature Optimization (Преждевременная оптимизация):**
   **Проблема:** Сложные оптимизации вводятся без реальной необходимости.
   **Решение:** Паттерн применяется только при доказанной проблеме с памятью.

4. **Data Clumps (Скопления данных):**
   **Проблема:** Одинаковые группы данных дублируются в разных объектах.
   **Решение:** Выделение повторяющихся данных в разделяемые объекты.

5. **Blob (Блоб):**
   **Проблема:** Один класс содержит слишком много несвязанных данных.
   **Решение:** Разделение на внутреннее (разделяемое) и внешнее (уникальное) состояние.

---

# Когда Использовать Flyweight?
1. **Большое количество объектов:** Когда приложение создает огромное число мелких объектов
2. **Ограниченные ресурсы:** На устройствах с ограниченной памятью (мобильные, embedded)
3. **Дублирование данных:** Когда объекты имеют значительную часть одинаковых данных
4. **Неизменяемое состояние:** Когда часть состояния объектов можно сделать неизменяемой

**Типичные применения:**
- Текстовые процессоры (оптимизация хранения символов)
- Графические движки (оптимизация частиц, деревьев, тайлов)
- Игровые миры (оптимизация NPC с одинаковыми характеристиками)
- UI библиотеки (разделение общих стилей элементов)

---

# Ограничения Паттерна:
1. **Сложность:** Увеличивает сложность кода из-за разделения состояния
2. **Потокобезопасность:** Требует аккуратной реализации при работе в многопоточной среде
3. **Не для всех случаев:** Эффективен только при наличии значительного дублирования данных
4. **Поиск баланса:** Важно правильно разделить внутреннее и внешнее состояние