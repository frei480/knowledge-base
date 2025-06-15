---
tags:
  - АрхитектураПО
---
# Паттерн Bridge (Мост)
**Назначение:** Разделяет абстракцию и реализацию, позволяя им изменяться независимо. Заменяет наследование композицией, избегая взрывного роста классов при комбинации функциональностей.

**Структура:**
- **Абстракция (Abstraction):** Высокоуровневый интерфейс для клиента.
- **Расширенная абстракция (Refined Abstraction):** Уточнённые версии абстракции.
- **Реализация (Implementor):** Интерфейс для низкоуровневых операций.
- **Конкретная реализация (Concrete Implementor):** Реализация интерфейса Implementor.

---

# Пример На Python
```python
from abc import ABC, abstractmethod

# Реализация (Implementor)
class Renderer(ABC):
    @abstractmethod
    def render_shape(self, shape):
        pass

# Конкретные реализации (Concrete Implementors)
class VectorRenderer(Renderer):
    def render_shape(self, shape):
        return f"Рисуем {shape} векторно"

class RasterRenderer(Renderer):
    def render_shape(self, shape):
        return f"Рисуем {shape} растрово"

# Абстракция (Abstraction)
class Shape:
    def __init__(self, renderer):
        self.renderer = renderer
    
    def draw(self):
        pass

# Расширенные абстракции (Refined Abstractions)
class Circle(Shape):
    def draw(self):
        return self.renderer.render_shape("круг")

class Square(Shape):
    def draw(self):
        return self.renderer.render_shape("квадрат")

# Клиентский код
vector = VectorRenderer()
raster = RasterRenderer()

circle = Circle(vector)
square = Square(raster)

print(circle.draw())  # Рисуем круг векторно
print(square.draw())  # Рисуем квадрат растрово
```

---

# Пример На C#
```csharp
using System;

// Реализация (Implementor)
public interface IRenderer
{
    string RenderShape(string shape);
}

// Конкретные реализации (Concrete Implementors)
public class VectorRenderer : IRenderer
{
    public string RenderShape(string shape) => $"Рисуем {shape} векторно";
}

public class RasterRenderer : IRenderer
{
    public string RenderShape(string shape) => $"Рисуем {shape} растрово";
}

// Абстракция (Abstraction)
public abstract class Shape
{
    protected IRenderer renderer;
    protected Shape(IRenderer renderer) => this.renderer = renderer;
    public abstract string Draw();
}

// Расширенные абстракции (Refined Abstractions)
public class Circle : Shape
{
    public Circle(IRenderer renderer) : base(renderer) { }
    public override string Draw() => renderer.RenderShape("круг");
}

public class Square : Shape
{
    public Square(IRenderer renderer) : base(renderer) { }
    public override string Draw() => renderer.RenderShape("квадрат");
}

// Клиентский код
class Program
{
    static void Main()
    {
        IRenderer vector = new VectorRenderer();
        IRenderer raster = new RasterRenderer();

        var circle = new Circle(vector);
        var square = new Square(raster);

        Console.WriteLine(circle.Draw());  // Рисуем круг векторно
        Console.WriteLine(square.Draw());  // Рисуем квадрат растрово
    }
}
```

---

# Против Каких Антипаттернов Применяется Bridge?
1. **Наследование вместо композиции:**
   - **Проблема:** Создание подклассов для каждой комбинации абстракции и реализации (например, `CircleVector`, `CircleRaster`).
   - **Решение:** Bridge заменяет наследование композицией, разделяя оси изменений.

2. **Взрывной рост классов (Class Explosion):**
   - **Проблема:** При добавлении новой функциональности количество классов растёт экспоненциально.
   - **Решение:** Паттерн позволяет добавлять новые абстракции и реализации независимо.

3. **Жёсткая связь абстракции и реализации:**
   - **Проблема:** Изменение реализации требует модификации абстракции.
   - **Решение:** Реализация инкапсулирована в отдельной иерархии, абстракция делегирует ей работу.

---

**Когда использовать Bridge?**
- Когда нужно разделить монолитный класс на части, которые могут изменяться независимо.
- Если система должна поддерживать несколько реализаций (например, рендеринг, базы данных, ОС).
- Когда нужно переключать реализации во время выполнения.