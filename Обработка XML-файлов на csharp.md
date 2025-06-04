---
tags:
  - csharp
  - xml
---

Для работы с вложенными тегами вида `<attribute name="param1" value="2">` в C# с использованием `XDocument` и `XElement`, можно создать структуру данных и преобразовать её в XML. Вот подробное решение:

---

### **1. Создание XML с вложенными тегами `<attribute>`**
```csharp
using System.Xml.Linq;

// Создаем родительский элемент с коллекцией атрибутов
XElement root = new XElement("Parameters",
    new XElement("attribute",
        new XAttribute("name", "param1"),
        new XAttribute("value", 2)
    ),
    new XElement("attribute",
        new XAttribute("name", "param2"),
        new XAttribute("value", 25)
    )
);

// Сохраняем в файл
root.Save("attributes.xml");
```

**Результат (attributes.xml):**
```xml
<Parameters>
  <attribute name="param1" value="2" />
  <attribute name="param2" value="25" />
</Parameters>
```

---

### **2. Динамическое создание из коллекции (списка/словаря)**
Если атрибуты хранятся в коллекции:
```csharp
var parameters = new Dictionary<string, object>
{
    { "param1", 2 },
    { "param2", 25 },
    { "param3", "text" }
};

XElement dynamicRoot = new XElement("Parameters",
    parameters.Select(p => 
        new XElement("attribute",
            new XAttribute("name", p.Key),
            new XAttribute("value", p.Value)
        )
    )
);

dynamicRoot.Save("dynamic_attributes.xml");
```

---

### **3. Сериализация объекта в XML**
Создадим класс-модель и преобразуем его в XML:

#### **3.1. Класс-модель**
```csharp
public class ParameterCollection
{
    public List<ParameterAttribute> Attributes { get; set; } = new List<ParameterAttribute>();
}

public class ParameterAttribute
{
    public string Name { get; set; }
    public object Value { get; set; }
}
```

#### **3.2. Преобразование объекта в XML**
```csharp
var parameters = new ParameterCollection
{
    Attributes = new List<ParameterAttribute>
    {
        new ParameterAttribute { Name = "param1", Value = 2 },
        new ParameterAttribute { Name = "param2", Value = 25 }
    }
};

XElement serializedXml = new XElement("Parameters",
    parameters.Attributes.Select(a =>
        new XElement("attribute",
            new XAttribute("name", a.Name),
            new XAttribute("value", a.Value)
        )
    )
);

serializedXml.Save("serialized.xml");
```

---

### **4. Чтение XML обратно в объект**
```csharp
XDocument doc = XDocument.Load("serialized.xml");
var loadedParameters = new ParameterCollection
{
    Attributes = doc.Root
        .Elements("attribute")
        .Select(e => new ParameterAttribute
        {
            Name = (string)e.Attribute("name"),
            Value = ParseValue((string)e.Attribute("value")) // Конвертация типов
        })
        .ToList()
};

// Метод для преобразования строки в нужный тип данных
private static object ParseValue(string value)
{
    if (int.TryParse(value, out int intValue)) return intValue;
    if (double.TryParse(value, out double doubleValue)) return doubleValue;
    return value; // Возвращаем как строку, если не число
}
```

---

### **5. Обработка сложных случаев**
#### **5.1. Вложенные контейнеры**
Если атрибуты находятся внутри тега `<attributes>`:
```csharp
XElement container = new XElement("Config",
    new XElement("attributes",
        new XElement("attribute", new XAttribute("name", "param1"), new XAttribute("value", 2)),
        new XElement("attribute", new XAttribute("name", "param2"), new XAttribute("value", 25))
    )
);
```

#### **5.2. Проверка существования атрибутов**
```csharp
foreach (XElement attrElement in doc.Descendants("attribute"))
{
    string name = attrElement.Attribute("name")?.Value ?? "unknown";
    string value = attrElement.Attribute("value")?.Value ?? "default";
    
    if (name == "unknown" || value == "default")
    {
        // Логирование ошибки
    }
}
```

---

### **6. Советы**
1. **Типы данных**: Атрибуты XML всегда строки. Используйте преобразование (например, `int.Parse`) при чтении.
2. **Пространства имен**: Если нужно, добавьте их через `XNamespace`.
3. **Валидация**: Используйте XSD для проверки структуры XML.
4. **Обработка ошибок**: Оберните код в `try-catch` для обработки исключений.

---

### **Итоговый пример**
```csharp
using System.Xml.Linq;

// Создание
XElement config = new XElement("Configuration",
    new XElement("Attributes",
        new XElement("attribute", new XAttribute("name", "maxThreads"), new XAttribute("value", 4)),
        new XElement("attribute", new XAttribute("name", "timeout"), new XAttribute("value", "30s"))
    )
);

// Сохранение
config.Save("app_config.xml");

// Чтение
var settings = XDocument.Load("app_config.xml")
    .Descendants("attribute")
    .ToDictionary(
        e => (string)e.Attribute("name"),
        e => (string)e.Attribute("value")
    );

int maxThreads = int.Parse(settings["maxThreads"]);
string timeout = settings["timeout"];
```

Этот подход позволяет легко работать с XML-структурами любой сложности, используя LINQ to XML.