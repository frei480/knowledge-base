---
tags:
  - csharp
---

Для работы с методами сторонних DLL через **System.Reflection** в C# используется подход **позднего связывания** (late binding). Это полезно, когда:
- Вы не можете добавить прямую ссылку на DLL (например, она загружается динамически).
- Нужно исследовать содержимое DLL во время выполнения.
- Вы хотите вызывать методы, классы или получать информацию о типах без явного знания структуры библиотеки.

---

# **Шаги Для Использования Reflection С Управляемой DLL**

## 1. **Загрузите DLL**
Используйте `Assembly.LoadFrom` или `Assembly.LoadFile`:
```csharp
using System.Reflection;

// Загрузка сборки
Assembly myDll = Assembly.LoadFrom(@"C:\Путь\к\YourLibrary.dll");
```

## 2. **Получите Тип (класс) Из DLL**
```csharp
// Получить тип по полному имени (Namespace + ClassName)
Type targetType = myDll.GetType("YourLibrary.Namespace.ClassName");

// Пример: если класс называется "MathHelper" в неймспейсе "Calculator"
// Type mathType = myDll.GetType("Calculator.MathHelper");
```

## 3. **Создайте Экземпляр класса**
```csharp
// Создание объекта через Activator
object instance = Activator.CreateInstance(targetType);

// Если у класса есть конструктор с параметрами:
// object instance = Activator.CreateInstance(targetType, аргументы);
```

## 4. **Получите Метод И Вызовите его**
```csharp
// Получить метод по имени и параметрам
MethodInfo method = targetType.GetMethod("MethodName");

// Пример для метода Add(int a, int b)
MethodInfo addMethod = targetType.GetMethod("Add", new Type[] { typeof(int), typeof(int) });

// Вызов метода
object result = addMethod.Invoke(instance, new object[] { 5, 3 });
Console.WriteLine($"Результат: {result}"); // 8
```

---

# **Пример: Полный код**
```csharp
using System;
using System.Reflection;

public class Program
{
    static void Main()
    {
        // 1. Загрузка DLL
        Assembly dll = Assembly.LoadFrom("MathLibrary.dll");

        // 2. Получение типа
        Type mathType = dll.GetType("MathLibrary.MathOperations");

        // 3. Создание экземпляра
        object mathInstance = Activator.CreateInstance(mathType);

        // 4. Получение метода "Add"
        MethodInfo addMethod = mathType.GetMethod("Add", 
            new Type[] { typeof(int), typeof(int) });

        // 5. Вызов метода
        object result = addMethod.Invoke(mathInstance, new object[] { 10, 20 });
        Console.WriteLine($"10 + 20 = {result}"); // 30
    }
}
```

---

# **Особые случаи**

## 1. **Вызов Статического метода**
Если метод статический, передавайте `null` вместо экземпляра:
```csharp
MethodInfo staticMethod = targetType.GetMethod("StaticMethod");
staticMethod.Invoke(null, new object[] { параметры });
```

## 2. **Работа С Приватными методами**
Используйте `BindingFlags`:
```csharp
MethodInfo privateMethod = targetType.GetMethod("SecretMethod", 
    BindingFlags.NonPublic | BindingFlags.Instance);

privateMethod.Invoke(instance, new object[] { });
```

## 3. **Получение Всех Методов класса**
```csharp
foreach (MethodInfo method in targetType.GetMethods())
{
    Console.WriteLine($"Метод: {method.Name}");
    Console.WriteLine($"Параметры: {string.Join(", ", method.GetParameters())}");
}
```

## 4. **Работа С Параметрами out/ref**
```csharp
MethodInfo methodWithOutParam = targetType.GetMethod("TryParse");
object[] args = new object[] { "123", null }; // out параметр инициализируется как null
methodWithOutParam.Invoke(instance, args);
int parsedValue = (int)args[1]; // Получить значение из out
```

---

# **Обработка исключений**
- **FileNotFoundException**: DLL не найдена.
- **TypeLoadException**: Не удалось загрузить тип.
- **MissingMethodException**: Метод не существует.
- **TargetInvocationException**: Ошибка внутри вызванного метода.

```csharp
try
{
    MethodInfo method = targetType.GetMethod("SomeMethod");
    method.Invoke(instance, параметры);
}
catch (TargetInvocationException ex)
{
    Console.WriteLine($"Ошибка в методе: {ex.InnerException.Message}");
}
```

---

# **Плюсы И Минусы Reflection**
- **Плюсы**:
  - Динамическая загрузка библиотек.
  - Возможность интроспекции (анализа типов, методов, свойств).
- **Минусы**:
  - Медленнее, чем прямой вызов.
  - Нет проверки типов на этапе компиляции.
  - Риск ошибок времени выполнения.

---

# **Пример: Загрузка плагина**
```csharp
// Интерфейс для плагина
public interface IPlugin
{
    string Name { get; }
    void Execute();
}

// Динамическая загрузка плагина
Assembly pluginAssembly = Assembly.LoadFrom("MyPlugin.dll");
Type pluginType = pluginAssembly.GetTypes()
    .FirstOrDefault(t => typeof(IPlugin).IsAssignableFrom(t));

if (pluginType != null)
{
    IPlugin plugin = (IPlugin)Activator.CreateInstance(pluginType);
    Console.WriteLine($"Загружен плагин: {plugin.Name}");
    plugin.Execute();
}
```

---

# **Итог**
Reflection — мощный инструмент для динамической работы с DLL, но используйте его осторожно:
- Применяйте только там, где нельзя обойтись прямыми ссылками.
- Всегда обрабатывайте исключения.
- Проверяйте существование типов и методов перед вызовом.