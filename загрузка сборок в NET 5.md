Загрузка сборок в .NET 5.0 (и более новых версиях) осуществляется через API `AssemblyLoadContext`, который предоставляет гибкий контроль над загрузкой и выгрузкой сборок. Вот основные методы и примеры:

---

### **1. Использование `AssemblyLoadContext` (рекомендуемый способ)**
Позволяет загружать сборки в изолированный контекст, который можно выгрузить.

```csharp
using System;
using System.Reflection;
using System.Runtime.Loader;

public class PluginLoadContext : AssemblyLoadContext
{
    public PluginLoadContext() : base(isCollectible: true) { }

    protected override Assembly? Load(AssemblyName assemblyName)
    {
        // Возвращаем null, чтобы использовать стандартную логику загрузки
        return null;
    }
}

// Пример загрузки сборки
var loadContext = new PluginLoadContext();
Assembly assembly = loadContext.LoadFromAssemblyPath(@"C:\Path\To\Your\Assembly.dll");

// Выгрузка контекста (сборки будут выгружены)
loadContext.Unload();
```

---

### **2. Использование `Assembly.LoadFrom`**
Загружает сборку по указанному пути, но может вызвать проблемы с дублированием сборок.

```csharp
Assembly assembly = Assembly.LoadFrom(@"C:\Path\To\Your\Assembly.dll");
```

---

### **3. Использование `Assembly.LoadFile`**
Загружает сборку как изолированный модуль (не разрешает зависимости автоматически).

```csharp
Assembly assembly = Assembly.LoadFile(@"C:\Path\To\Your\Assembly.dll");
```

---

### **Пример: Загрузка и использование типа из сборки**
```csharp
// Загрузка сборки
Assembly pluginAssembly = Assembly.LoadFrom("MyPlugin.dll");

// Получение типа по имени
Type? pluginType = pluginAssembly.GetType("MyPlugin.MainClass");

if (pluginType != null)
{
    // Создание экземпляра
    object? pluginInstance = Activator.CreateInstance(pluginType);

    // Вызов метода
    MethodInfo? method = pluginType.GetMethod("Execute");
    method?.Invoke(pluginInstance, null);
}
```

---

### **Важные замечания**
1. **Зависимости**  
   Если загружаемая сборка зависит от других DLL, убедитесь, что они доступны в каталоге приложения или в путях, указанных в `AssemblyLoadContext.Resolving`.

2. **Выгрузка сборок**  
   Чтобы выгрузить сборки, используйте `AssemblyLoadContext.Unload()`. Сборки, загруженные через `LoadFrom`/`LoadFile`, нельзя выгрузить без выгрузки домена (что недоступно в .NET Core/5+).

3. **Изоляция**  
   Для плагинов создавайте отдельный `AssemblyLoadContext`, чтобы избежать конфликтов версий.

---

### **Обработка зависимостей**
Если сборка требует дополнительные зависимости, можно переопределить логику загрузки:
```csharp
public class CustomLoadContext : AssemblyLoadContext
{
    protected override Assembly Load(AssemblyName assemblyName)
    {
        // Попытка загрузить зависимость из текущего контекста
        var defaultContext = AssemblyLoadContext.Default;
        Assembly? assembly = defaultContext.Assemblies
            .FirstOrDefault(a => a.GetName().Name == assemblyName.Name);
        
        return assembly ?? base.Load(assemblyName);
    }
}
```

---

### **Совет**
Для сложных сценариев (например, плагинов) используйте библиотеку `Microsoft.Extensions.DependencyModel` для анализа зависимостей. Это поможет корректно загружать все необходимые DLL.
[[как использовать DependencyModel для управления зависимостями NET 5]]