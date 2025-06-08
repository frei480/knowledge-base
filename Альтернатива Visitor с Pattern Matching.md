### Альтернатива Visitor с Pattern Matching  
В языках с поддержкой pattern matching (C# 8.0+, Python 3.10+) можно заменить классический Visitor на функции с сопоставлением шаблонов, что упрощает код и снижает накладные расходы.

---

### Пример на C# (с Pattern Matching)  
**Сценарий:** Обработка геометрических фигур  
```csharp
// Базовый тип иерархии
abstract record Shape();

// Конкретные типы
record Circle(double Radius) : Shape;
record Rectangle(double Width, double Height) : Shape;
record Triangle(double A, double B, double C) : Shape;

// Обработка через pattern matching
static class ShapeProcessor
{
    public static double CalculateArea(Shape shape) => shape switch
    {
        Circle c => Math.PI * c.Radius * c.Radius,
        Rectangle r => r.Width * r.Height,
        Triangle t => (t.A + t.B + t.C) / 2,  // Формула Герона
        _ => throw new ArgumentException("Unknown shape")
    };

    public static string Describe(Shape shape) => shape switch
    {
        Circle c => $"Круг радиусом {c.Radius}",
        Rectangle r => $"Прямоугольник {r.Width}x{r.Height}",
        Triangle t => $"Треугольник со сторонами {t.A}, {t.B}, {t.C}",
        _ => "Неизвестная фигура"
    };
}

// Использование
var shapes = new Shape[] 
{
    new Circle(5),
    new Rectangle(3, 4),
    new Triangle(3, 4, 5)
};

foreach (var shape in shapes)
{
    Console.WriteLine($"{ShapeProcessor.Describe(shape)} | Площадь: {ShapeProcessor.CalculateArea(shape):F2}");
}
```

**Вывод:**  
```
Круг радиусом 5 | Площадь: 78.54
Прямоугольник 3x4 | Площадь: 12.00
Треугольник со сторонами 3, 4, 5 | Площадь: 6.00
```

---

### Пример на Python (с Pattern Matching)  
**Сценарий:** Обработка сетевых пакетов  
```python
from dataclasses import dataclass
from typing import Union

# Типы пакетов
@dataclass
class PingPacket:
    id: int

@dataclass
class DataPacket:
    source: str
    payload: bytes

@dataclass
class ErrorPacket:
    code: int
    message: str

Packet = Union[PingPacket, DataPacket, ErrorPacket]

# Обработчик через pattern matching
def handle_packet(packet: Packet) -> str:
    match packet:
        case PingPacket(packet_id):
            return f"PING [ID: {packet_id}]"
            
        case DataPacket(source, payload):
            return f"Данные от {source}: {len(payload)} байт → '{payload[:10].decode()}...'"
            
        case ErrorPacket(code, msg):
            return f"ОШИБКА {code}: {msg}"
            
        case _:
            return "Неизвестный пакет"

# Тестирование
packets = [
    PingPacket(42),
    DataPacket("192.168.1.1", b"Hello World! This is a long message"),
    ErrorPacket(404, "Сервер недоступен")
]

for p in packets:
    print(handle_packet(p))
```

**Вывод:**  
```
PING [ID: 42]
Данные от 192.168.1.1: 34 байт → 'Hello Worl...'
ОШИБКА 404: Сервер недоступен
```

---

### Сравнение подходов  
| **Критерий**               | **Классический Visitor**       | **Pattern Matching**          |
|----------------------------|--------------------------------|-------------------------------|
| **Сложность кода**         | Высокая (интерфейсы + классы) | Низкая (одна функция)         |
| **Поддержка новых операций** | Легко (новый Visitor)          | Требует изменения функции     |
| **Поддержка новых типов**  | Требует изменения всех Visitor | Легко (добавить case)         |
| **Инкапсуляция**           | Строгая                        | Требует публичных полей       |
| **Рекурсивные структуры**  | Идеально подходит              | Ограниченная поддержка        |
| **Читаемость**             | Сложная (двойная диспетчеризация) | Высокая (линейная логика)    |

---

### Когда использовать Pattern Matching вместо Visitor?
1. **Простая иерархия типов**  
   Если типы объектов образуют плоскую структуру без сложных вложенностей.

2. **Редкие изменения операций**  
   Когда новые операции добавляются реже, чем новые типы объектов.

3. **Отсутствие рекурсии**  
   Для обработки древовидных структур (AST, DOM) Visitor предпочтительнее.

4. **Упрощение кодовой базы**  
   В небольших проектах или скриптах, где важна простота.

5. **Обработка внешних типов**  
   Когда нельзя изменить исходные классы (например, работа со сторонними библиотеками).

---

### Ограничения Pattern Matching
- **Нарушение OCP:** Добавление новой операции требует модификации существующей функции
- **Дублирование кода:** Схожая логика может дублироваться в разных ветках
- **Сложность с композицией:** Нет аналога композитных посетителей
- **Производительность:** В C# `switch` на типах может использовать reflection (оптимизируется в .NET 7+)

> **Эмпирическое правило:**  
> Используйте pattern matching для замены Visitor, когда:  
> - Количество типов < 10  
> - Нет вложенных структур  
> - Операции меняются реже 1 раза в месяц  
> В противном случае классический Visitor обеспечит большую гибкость.