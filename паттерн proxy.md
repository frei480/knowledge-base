## Паттерн "Представитель" (Proxy)

**Паттерн "Представитель" (Proxy)** — структурный паттерн, предоставляющий объект-заменитель для контроля доступа к другому объекту. Прокси действует как оболочка, добавляя дополнительную логику (ленивую инициализацию, кэширование, контроль доступа) без изменения основного объекта.

### Типы прокси:
1. **Виртуальный** (ленивая инициализация)
2. **Защищающий** (контроль прав доступа)
3. **Удалённый** (работа с сетевыми ресурсами)
4. **Кэширующий** (сохранение результатов)
5. **Логирующий** (журналирование операций)

### Компоненты:
1. **Интерфейс Subject** - общий интерфейс для RealSubject и Proxy
2. **RealSubject** - реальный объект
3. **Proxy** - объект-представитель

---

### Пример на Python (Виртуальный прокси)
```python
from abc import ABC, abstractmethod

# Интерфейс
class Database(ABC):
    @abstractmethod
    def query(self, sql: str): pass

# RealSubject
class RealDatabase(Database):
    def query(self, sql: str):
        print(f"Выполняю тяжёлый запрос: {sql}")
        return f"Результат {sql}"

# Proxy (ленивая инициализация)
class LazyDatabaseProxy(Database):
    def __init__(self):
        self._real_db = None
    
    def query(self, sql: str):
        if self._real_db is None:
            print("Создаю подключение к БД...")
            self._real_db = RealDatabase()
        return self._real_db.query(sql)

# Клиентский код
db = LazyDatabaseProxy()
print(db.query("SELECT * FROM users"))  # Создаёт подключение
print(db.query("SELECT * FROM orders")) # Использует существующее
```

### Пример на C# (Защищающий прокси)
```csharp
using System;

// Интерфейс
public interface IDocument {
    void Display();
}

// RealSubject
class ConfidentialDocument : IDocument {
    public void Display() {
        Console.WriteLine("Отображаю секретный документ");
    }
}

// Proxy (контроль доступа)
class ProtectedDocumentProxy : IDocument {
    private ConfidentialDocument _realDoc;
    private string _userRole;
    
    public ProtectedDocumentProxy(string userRole) {
        _userRole = userRole;
    }
    
    public void Display() {
        if (_userRole == "Admin") {
            if (_realDoc == null) 
                _realDoc = new ConfidentialDocument();
            
            _realDoc.Display();
        }
        else {
            Console.WriteLine("Ошибка: Недостаточно прав!");
        }
    }
}

// Клиентский код
class Program {
    static void Main() {
        IDocument doc = new ProtectedDocumentProxy("User");
        doc.Display(); // Ошибка прав
        
        IDocument adminDoc = new ProtectedDocumentProxy("Admin");
        adminDoc.Display(); // Успешный доступ
    }
}
```

---

## Против каких антипаттернов борется Proxy?

### 1. **Нарушение принципа единственной ответственности (SRP)**
**Проблема:** Класс совмещает бизнес-логику с дополнительными задачами (кэширование, доступ и т.д.)  
**Решение:** Прокси выносит вспомогательную функциональность в отдельный класс.

**Пример нарушения:**
```python
class ReportGenerator:
    def generate_report(self):
        if not self._check_access():  # Смешивание логики
            raise PermissionError()
        # Генерация отчёта
```

### 2. **Прямой доступ к ресурсоёмким объектам**
**Проблема:** Клиенты напрямую создают дорогие объекты, даже когда они не нужны.  
**Решение:** Виртуальный прокси реализует ленивую инициализацию.

**Пример нарушения:**
```csharp
// Клиент всегда создаёт тяжёлый объект
var db = new RealDatabase(); // Создаётся сразу
if (condition) db.Query(...);
```

### 3. **Размазывание кода контроля доступа**
**Проблема:** Проверки прав дублируются во множестве мест кода.  
**Решение:** Защищающий прокси централизует контроль доступа.

**Пример нарушения:**
```python
class ServiceA:
    def operation(self, user):
        if not user.is_admin:  # Дублирование
            raise Error()
        
class ServiceB:
    def action(self, user):
        if not user.is_admin:  # Дублирование
            raise Error()
```

### 4. **Жёсткая привязка к удалённым ресурсам**
**Проблема:** Клиент зависит от сетевых вызовов напрямую.  
**Решение:** Удалённый прокси инкапсулирует работу с сетью.

**Пример нарушения:**
```csharp
// Клиент напрямую работает с сетевым API
var result = await httpClient.GetAsync("https://api.com/data");
```

### 5. **Отсутствие кэширования**
**Проблема:** Повторные запросы к медленным сервисам без кэширования.  
**Решение:** Кэширующий прокси сохраняет результаты запросов.

**Пример нарушения:**
```python
def get_data(id):
    # Всегда обращается к БД
    return db.query("SELECT ...")  # Медленно при повторах
```

### 6. **Нарушение открытости/закрытости (OCP)**
**Проблема:** Для добавления новой функциональности требуется менять существующий класс.  
**Решение:** Прокси добавляет новое поведение без изменения исходного объекта.

---

### Преимущества использования Proxy:
1. **Контроль жизненного цикла** объекта (особенно для ресурсоёмких объектов)
2. **Добавление функциональности** без изменения основного кода
3. **Упрощение клиентского кода** (клиент не знает о сложностях доступа)
4. **Повышение безопасности** через централизованный контроль
5. **Оптимизация производительности** (кэширование, ленивая загрузка)

### Когда применять:
- Работа с ресурсоёмкими объектами (БД, сетевые соединения)
- Контроль прав доступа
- Кэширование результатов операций
- Упрощение работы с удалёнными системами
- Журналирование операций