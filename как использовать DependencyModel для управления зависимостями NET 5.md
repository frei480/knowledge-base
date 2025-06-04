---
tags:
  - csharp
---

**DependencyModel** — это часть экосистемы .NET (Core/5+), которая предоставляет API для работы с метаданными зависимостей приложения. Она позволяет анализировать зависимости проекта, включая ссылки на сборки, пакеты NuGet, runtime-активы и другие ресурсы. Основная цель — упростить управление зависимостями в динамических сценариях (например, загрузка плагинов).

---

### **Основные компоненты**
1. **`DependencyContext`**  
   Главный класс, содержащий информацию о зависимостях приложения. Данные берутся из файла **`<app>.deps.json`**, который генерируется при сборке проекта.

2. **`DependencyContext.Default`**  
   Статическое свойство, возвращающее контекст зависимостей текущего приложения.

3. **`CompilationLibrary` и `RuntimeLibrary`**  
   Классы, описывающие зависимости:
   - `CompilationLibrary` — сборки, необходимые для компиляции.
   - `RuntimeLibrary` — сборки, требуемые во время выполнения.

---

### **Как получить DependencyContext**
#### 1. Для текущего приложения:
```csharp
using Microsoft.Extensions.DependencyModel;

DependencyContext context = DependencyContext.Default;
```

#### 2. Для произвольной сборки:
```csharp
using System.Reflection;
using Microsoft.Extensions.DependencyModel;

Assembly assembly = Assembly.GetEntryAssembly(); // или другая сборка
DependencyContext context = DependencyContext.Load(assembly);
```

---

### **Примеры использования**
#### 1. Получение списка runtime-зависимостей:
```csharp
foreach (var runtimeLib in context.RuntimeLibraries)
{
    Console.WriteLine($"Library: {runtimeLib.Name}, Version: {runtimeLib.Version}");
}
```

#### 2. Поиск путей к файлам сборок:
```csharp
var runtimeAssemblies = context.RuntimeLibraries
    .SelectMany(lib => lib.GetDefaultAssemblyNames(context))
    .ToList();

foreach (var assemblyName in runtimeAssemblies)
{
    Console.WriteLine(assemblyName.FullName);
}
```

---

### **Практический пример: Загрузка зависимостей плагина**
Предположим, вы хотите загрузить плагин, который зависит от тех же пакетов, что и основное приложение. Используйте `DependencyContext`, чтобы найти нужные сборки:

```csharp
public class PluginLoadContext : AssemblyLoadContext
{
    private readonly DependencyContext _dependencyContext;

    public PluginLoadContext(DependencyContext context) : base(isCollectible: true)
    {
        _dependencyContext = context;
    }

    protected override Assembly Load(AssemblyName assemblyName)
    {
        // Ищем сборку в зависимостях
        var library = _dependencyContext.RuntimeLibraries
            .FirstOrDefault(lib => lib.Name.Equals(assemblyName.Name));

        if (library != null)
        {
            // Путь к сборке (например, из папки приложения)
            var assemblyPath = // ... логика поиска пути на основе library
            return LoadFromAssemblyPath(assemblyPath);
        }

        return null;
    }
}
```

---

### **Обработка конфликтов версий**
Если плагин требует версию зависимости, отличную от версии основного приложения, DependencyModel поможет определить это:
```csharp
var requiredLibrary = context.RuntimeLibraries
    .FirstOrDefault(lib => lib.Name == "Newtonsoft.Json" && lib.Version == "13.0.0");

if (requiredLibrary != null)
{
    // Загрузите конкретную версию из нужного места
}
```

---

### **Важные замечания**
1. **Файл `deps.json`**  
   DependencyContext работает только если файл `<app>.deps.json` существует. Он автоматически генерируется для:
   - Приложений, опубликованных через `dotnet publish`.
   - Не self-contained приложений.

   **Внимание:** В self-contained приложениях или при публикации с `<PublishSingleFile>true</PublishSingleFile>` файл `deps.json` может отсутствовать.

2. **Ограничения**  
   - DependencyContext не отслеживает зависимости динамически загружаемых сборок (только основного приложения).
   - Для плагинов может потребоваться отдельный анализ их `deps.json`.

---

### **Советы**
1. **Анализ зависимостей плагина**  
   Если плагин — это отдельная сборка, можно попытаться загрузить её `DependencyContext` через манифест:
   ```csharp
   var pluginAssembly = AssemblyLoadContext.Default.LoadFromAssemblyPath("Plugin.dll");
   var pluginContext = DependencyContext.Load(pluginAssembly);
   ```

2. **Использование `AssemblyDependencyResolver`**  
   В .NET Core 3.0+ появился класс `AssemblyDependencyResolver`, который упрощает разрешение зависимостей для плагинов:
   ```csharp
   var resolver = new AssemblyDependencyResolver("Plugin.dll");
   string? assemblyPath = resolver.ResolveAssemblyToPath(new AssemblyName("Newtonsoft.Json"));
   ```

---

### **Когда использовать DependencyModel?**
- **Динамическая загрузка плагинов**: Чтобы корректно разрешать зависимости.
- **Анализ окружения**: Получение списка всех зависимостей приложения.
- **Отладка конфликтов версий**: Поиск причин `FileNotFoundException` или `TypeLoadException`.

DependencyModel — мощный инструмент для сложных сценариев загрузки сборок, но требует аккуратного подхода при работе с версиями и путями.