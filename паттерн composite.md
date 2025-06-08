---
tags:
  - АрхитектураПО
---
## Паттерн "Компоновщик" (Composite)

**Компоновщик** — структурный паттерн, позволяющий сгруппировать объекты в древовидные структуры и работать с ними как с единым объектом. Он создаёт иерархию "часть-целое", где как отдельные объекты, так и их композиции обрабатываются единообразно.

### Ключевые особенности:
1. **Единый интерфейс** для простых и составных компонентов
2. **Рекурсивная композиция**: Компоненты могут содержать другие компоненты
3. **Прозрачность**: Клиентский код одинаково работает с простыми и составными объектами

### Компоненты:
1. **Компонент (Component)** - общий интерфейс
2. **Лист (Leaf)** - простой неделимый элемент
3. **Компоновщик (Composite)** - составной элемент, содержащий дочерние компоненты

---

### Пример на Python (Файловая система)
```python
from abc import ABC, abstractmethod
from typing import List

# Интерфейс компонента
class FileSystemItem(ABC):
    @abstractmethod
    def get_size(self) -> int:
        pass

# Лист (файл)
class File(FileSystemItem):
    def __init__(self, name: str, size: int):
        self.name = name
        self.size = size
    
    def get_size(self) -> int:
        return self.size

# Компоновщик (папка)
class Folder(FileSystemItem):
    def __init__(self, name: str):
        self.name = name
        self.children: List[FileSystemItem] = []
    
    def add(self, item: FileSystemItem):
        self.children.append(item)
    
    def remove(self, item: FileSystemItem):
        self.children.remove(item)
    
    def get_size(self) -> int:
        total = 0
        for child in self.children:
            total += child.get_size()
        return total

# Клиентский код
if __name__ == "__main__":
    # Создаем файлы
    file1 = File("document.txt", 150)
    file2 = File("image.jpg", 1200)
    
    # Создаем подпапку
    subfolder = Folder("Subfolder")
    subfolder.add(File("notes.md", 300))
    
    # Создаем основную папку
    root = Folder("Root")
    root.add(file1)
    root.add(file2)
    root.add(subfolder)
    
    # Вычисляем общий размер
    print(f"Total size: {root.get_size()} bytes")  # 150 + 1200 + 300 = 1650
```

### Пример на C# (Графический редактор)
```csharp
using System;
using System.Collections.Generic;

// Интерфейс компонента
public interface IGraphic
{
    void Draw();
    void Move(int x, int y);
}

// Лист (простая фигура)
public class Circle : IGraphic
{
    public void Draw() => Console.WriteLine("Рисую круг");
    public void Move(int x, int y) => Console.WriteLine($"Перемещаю круг на ({x}, {y})");
}

// Лист (простая фигура)
public class Square : IGraphic
{
    public void Draw() => Console.WriteLine("Рисую квадрат");
    public void Move(int x, int y) => Console.WriteLine($"Перемещаю квадрат на ({x}, {y})");
}

// Компоновщик (группа фигур)
public class GraphicGroup : IGraphic
{
    private readonly List<IGraphic> _children = new List<IGraphic>();
    
    public void Add(IGraphic graphic) => _children.Add(graphic);
    public void Remove(IGraphic graphic) => _children.Remove(graphic);
    
    public void Draw()
    {
        Console.WriteLine("Начинаю рисовать группу:");
        foreach (var child in _children)
        {
            child.Draw();
        }
    }
    
    public void Move(int x, int y)
    {
        Console.WriteLine($"Перемещаю группу на ({x}, {y}):");
        foreach (var child in _children)
        {
            child.Move(x, y);
        }
    }
}

class Program
{
    static void Main()
    {
        // Создаем фигуры
        var circle = new Circle();
        var square = new Square();
        
        // Создаем группу
        var group1 = new GraphicGroup();
        group1.Add(circle);
        group1.Add(square);
        
        // Создаем вторую группу
        var group2 = new GraphicGroup();
        group2.Add(new Square());
        group2.Add(new Circle());
        
        // Создаем корневую группу
        var rootGroup = new GraphicGroup();
        rootGroup.Add(group1);
        rootGroup.Add(group2);
        
        // Работаем с композицией
        rootGroup.Draw();
        rootGroup.Move(10, 20);
    }
}
```

---

## Против каких антипаттернов борется Composite?

### 1. **Жёсткая иерархия объектов**
**Проблема:** Разделение кода для обработки одиночных объектов и групп  
**Решение:** Единый интерфейс для всех компонентов

```python
# Антипаттерн
def process_item(item):
    if isinstance(item, File):
        # обработка файла
    elif isinstance(item, Folder):
        # обработка папки (с рекурсией)
```

### 2. **Нарушение Open-Closed Principle**
**Проблема:** Добавление новых типов требует изменения существующего кода  
**Решение:** Новые компоненты добавляются без изменения клиентского кода

### 3. **Дублирование кода обхода**
**Проблема:** Повторение кода для рекурсивного обхода в разных местах  
**Решение:** Логика обхода инкапсулирована в компонентах

```csharp
// Без Composite
public void ProcessGroup(Group group) {
    foreach (var item in group.Items) {
        if (item is Shape) ProcessShape(item);
        else if (item is Group) ProcessGroup(item); // Рекурсия
    }
}

// С Composite
group.Process(); // Рекурсия внутри компонента
```

### 4. **Смешивание бизнес-логики и обхода**
**Проблема:** Код обработки содержит логику обхода структуры  
**Решение:** Компоненты сами управляют обходом своих дочерних элементов

### 5. **Ограниченность древовидных структур**
**Проблема:** Невозможность создания сложных иерархий  
**Решение:** Рекурсивная композиция любой сложности

### 6. **Несогласованные интерфейсы**
**Проблема:** Разные методы для работы с одиночными объектами и группами  
**Решение:** Единый API для всех компонентов

---

### Преимущества использования Composite:
1. **Упрощение клиентского кода** - единое взаимодействие с простыми и сложными объектами
2. **Гибкость структуры** - легко добавлять новые типы компонентов
3. **Масштабируемость** - работа с древовидными структурами любой сложности
4. **Соблюдение принципов [[SOLID принципы|SOLID]]**:
   - Open/Closed (новые компоненты без изменений)
   - Single Responsibility (разделение обязанностей)

### Недостатки:
1. **Слишком общий дизайн** - сложность сделать интерфейс подходящим для всех компонентов
2. **Безопасность типов** - клиент может попытаться выполнить недопустимую операцию для листа

### Когда использовать:
- Представление иерархий "часть-целое"
- Обработка древовидных структур данных
- Единообразная работа с отдельными объектами и группами
- Генерация сложных UI-компонентов
- Обработка файловых систем

### Реальные примеры применения:
- GUI-фреймворки (виджеты и контейнеры)
- Файловые системы (файлы и директории)
- Документы (элементы и группы элементов)
- Системы аналитики (агрегация показателей)