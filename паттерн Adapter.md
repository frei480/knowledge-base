---
tags:
  - АрхитектураПО
---
## Паттерн "Адаптер" (Adapter)

**Адаптер** — структурный паттерн, позволяющий объектам с несовместимыми интерфейсами работать вместе. Он выступает мостом между двумя несовместимыми интерфейсами, преобразуя интерфейс одного класса в интерфейс, ожидаемый клиентом.

### Типы адаптеров:
1. **Адаптер класса** (наследование)
2. **Адаптер объекта** (композиция)

### Компоненты:
1. **Целевой интерфейс (Target)** - интерфейс, ожидаемый клиентом
2. **Адаптируемый класс (Adaptee)** - существующий класс с несовместимым интерфейсом
3. **Адаптер (Adapter)** - преобразует интерфейс Adaptee в Target

---

### Пример на Python (Адаптер объекта)
```python
# Целевой интерфейс
class EuropeanSocket:
    def voltage(self) -> int:
        return 230
    
    def live(self) -> int:
        return 1
    
    def neutral(self) -> int:
        return -1

# Адаптируемый класс (несовместимый интерфейс)
class USASocket:
    def voltage(self) -> int:
        return 120
    
    def live(self) -> int:
        return 1
    
    def neutral(self) -> int:
        return -1
    
    def ground(self) -> int:  # Дополнительный метод
        return 0

# Адаптер (композиция)
class USASocketAdapter(EuropeanSocket):
    def __init__(self, usa_socket: USASocket):
        self._usa_socket = usa_socket
    
    def voltage(self) -> int:
        return self._usa_socket.voltage()
    
    def live(self) -> int:
        return self._usa_socket.live()
    
    def neutral(self) -> int:
        return self._usa_socket.neutral()
    
    # Игнорируем ground(), чтобы соответствовать целевому интерфейсу

# Клиентский код
def charge_device(socket: EuropeanSocket):
    print(f"Charging at {socket.voltage()}V")
    print(f"Live: {socket.live()}, Neutral: {socket.neutral()}")

# Использование
european_socket = EuropeanSocket()
charge_device(european_socket)  # Работает напрямую

usa_socket = USASocket()
adapter = USASocketAdapter(usa_socket)
charge_device(adapter)  # Работает через адаптер
```

### Пример на C# (Адаптер класса)
```csharp
using System;

// Целевой интерфейс
public interface IJsonParser {
    string Parse(string data);
}

// Адаптируемый класс
public class XmlParser {
    public string ParseXml(string xml) {
        return $"<parsed>{xml}</parsed>";
    }
}

// Адаптер (наследование)
public class XmlToJsonAdapter : XmlParser, IJsonParser {
    public string Parse(string data) {
        // Преобразуем вызов JSON-метода в XML-логику
        string xmlResult = base.ParseXml(data);
        return ConvertXmlToJson(xmlResult);
    }

    private string ConvertXmlToJson(string xml) {
        return $"{{\"data\": \"{xml}\"}}"; // Упрощённое преобразование
    }
}

// Клиентский код
class Program {
    static void ProcessData(IJsonParser parser) {
        Console.WriteLine(parser.Parse("test data"));
    }

    static void Main() {
        // Использование адаптера
        IJsonParser adapter = new XmlToJsonAdapter();
        ProcessData(adapter);  // Работает с XML через JSON-интерфейс
        
        // Output: {"data": "<parsed>test data</parsed>"}
    }
}
```

---

## Против каких антипаттернов борется Adapter?

### 1. **Прямое использование несовместимых интерфейсов**
**Проблема:** Жёсткая зависимость от конкретной реализации  
**Решение:** Адаптер изолирует несовместимый интерфейс

```python
# Антипаттерн
class Client:
    def use_legacy(self):
        legacy = LegacySystem()
        data = legacy.get_data_in_xml()  # Прямое использование
        # ... конвертация в нужный формат
```

### 2. **Нарушение Open-Closed Principle**
**Проблема:** Модификация клиентского кода для поддержки новых интерфейсов  
**Решение:** Новые адаптеры добавляются без изменения клиента

```csharp
// Без адаптера
public void Process(ILegacyService service) {
    if (service is NewSystem newSystem) {
        // Специальная обработка
    }
}
```

### 3. **Дублирование кода преобразования**
**Проблема:** Повторение логики преобразования в разных местах  
**Решение:** Централизация преобразования в адаптере

```python
# Дублирование
class ServiceA:
    def process(self):
        data = legacy.get_xml()
        json = convert_xml_to_json(data)  # Та же логика в другом классе

class ServiceB:
    def execute(self):
        data = legacy.get_xml()
        json = convert_xml_to_json(data)  # Дубликат
```

### 4. **Нарушение принципа инверсии зависимостей**
**Проблема:** Зависимость от конкретных реализаций вместо абстракций  
**Решение:** Клиент зависит от целевого интерфейса

```csharp
// Нарушение DIP
public class ReportGenerator {
    private readonly XmlParser _parser;  // Зависимость от конкретного класса
    
    public ReportGenerator(XmlParser parser) {
        _parser = parser;
    }
}
```

### 5. **Усложнение тестирования**
**Проблема:** Невозможность изолировать тесты из-за жёстких зависимостей  
**Решение:** Адаптер позволяет подменять реальные объекты моками

```python
# Сложность тестирования
def test_processor():
    processor = DataProcessor(LegacySystem())  # Зависит от реальной системы
    # ... сложная настройка
    
# С адаптером
def test_processor():
    adapter = MockAdapter()  # Тестовый адаптер
    processor = DataProcessor(adapter)
```

### 6. **Загрязнение бизнес-логики**
**Проблема:** Смешивание основной логики с преобразованием данных  
**Решение:** Адаптер инкапсулирует преобразовательную логику

```csharp
// Смешивание логики
public class OrderService {
    public void CreateOrder(Order order) {
        // Основная логика
        var legacyFormat = ConvertToLegacyFormat(order);  // Преобразование
        _legacySystem.Process(legacyFormat);
    }
}
```

---

### Преимущества использования Adapter:
1. **Совместимость интерфейсов**: Интеграция несовместимых компонентов
2. **Принцип единой ответственности**: Разделение преобразования и бизнес-логики
3. **Повторное использование**: Возможность использовать существующие классы
4. **Гибкость**: Легкая замена адаптеров
5. **Тестируемость**: Упрощение изоляции компонентов

### Недостатки:
1. **Усложнение кода**: Добавление дополнительных классов
2. **Накладные расходы**: Дополнительный слой абстракции

### Когда использовать:
- Интеграция устаревших систем
- Работа со сторонними библиотеками
- Адаптация различных API
- Поддержка нескольких версий интерфейсов
- Упрощение тестирования

### Реальные примеры:
1. **Адаптеры баз данных**: Преобразование SQL-запросов к разным СУБД
2. **API-шлюзы**: Преобразование форматов данных (XML/JSON/Protobuf)
3. **Драйверы устройств**: Унифицированный интерфейс для оборудования
4. **Кросс-платформенные UI**: Адаптация нативных компонентов