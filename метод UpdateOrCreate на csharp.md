---
tags:
  - csharp
---

Для решения задачи создания или обновления объекта `Variable` на основе списка параметров можно использовать следующий подход:

```csharp
public class Variable
{
    public string StringValue { get; }
    public double? DoubleValue { get; }

    public Variable(string stringValue)
    {
        StringValue = stringValue;
        DoubleValue = null;
    }

    public Variable(double doubleValue)
    {
        DoubleValue = doubleValue;
        StringValue = null;
    }

    public bool IsDouble => DoubleValue.HasValue;
}

public static Variable UpdateOrCreate(Variable existingVariable, IEnumerable<object> parameters)
{
    double? lastDouble = null;
    string lastString = null;
    bool hasDouble = false;

    foreach (var param in parameters)
    {
        if (param == null) continue;

        if (param is double d)
        {
            lastDouble = d;
            hasDouble = true;
            lastString = null; // Сбрасываем строку, так как double имеет приоритет
        }
        else if (param is string s && !hasDouble)
        {
            lastString = s;
        }
    }

    if (lastDouble.HasValue)
    {
        return new Variable(lastDouble.Value);
    }
    else if (lastString != null)
    {
        return new Variable(lastString);
    }
    else
    {
        // Если параметров нет, возвращаем существующий объект или null
        return existingVariable;
    }
}
```

# Описание Метода:
1. **Класс `Variable`**:
   - Содержит свойства `StringValue` и `DoubleValue`.
   - Конструкторы гарантируют, что объект будет иметь значение только одного типа (string или double).
   - Свойство `IsDouble` указывает тип значения.

2. **Метод `UpdateOrCreate`**:
   - **Входные параметры**:
     - `existingVariable` — текущий объект (может быть `null`).
     - `parameters` — список параметров для обновления.
   - **Логика**:
     - Перебирает параметры, запоминая последние значения string и double.
     - Double имеет приоритет: если найден хотя бы один double, все последующие string игнорируются.
     - Если параметры содержат double, возвращается новый `Variable` с этим значением.
     - Если нет double, но есть string, возвращается `Variable` с этим string.
     - Если параметры отсутствуют или все null, возвращается `existingVariable`.

# Примеры Использования:
```csharp
// Пример 1: Создание нового объекта с double
var parameters1 = new object[] { null, "text", 3.14 };
Variable result1 = UpdateOrCreate(null, parameters1);
// result1: DoubleValue = 3.14

// Пример 2: Обновление существующего объекта на основе double
Variable existing2 = new Variable("old");
var parameters2 = new object[] { "new", 42.0 };
Variable result2 = UpdateOrCreate(existing2, parameters2);
// result2: DoubleValue = 42.0

// Пример 3: Параметры с string после double игнорируются
var parameters3 = new object[] { 5.0, "test" };
Variable result3 = UpdateOrCreate(null, parameters3);
// result3: DoubleValue = 5.0

// Пример 4: Возврат существующего объекта при отсутствии параметров
Variable existing4 = new Variable("existing");
Variable result4 = UpdateOrCreate(existing4, new object[] { });
// result4 == existing4
```

Этот метод обеспечивает обновление или создание объекта `Variable` на основе переданных параметров, учитывая приоритет типа double над string.