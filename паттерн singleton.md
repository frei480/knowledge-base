---
tags:
  - АрхитектураПО
---

# Паттерн Singleton (Одиночка)
**Назначение:**
Гарантирует, что класс имеет только один экземпляр, и предоставляет глобальную точку доступа к этому экземпляру.

## Ключевые Аспекты:
1. **Единственный экземпляр:** Конструктор скрыт, создание экземпляра контролируется изнутри класса.
2. **Глобальный доступ:** Статический метод для получения экземпляра.
3. **Потокобезопасность:** Особенно важно в многопоточных средах.

---

# Пример На Python
**Базовая реализация:**
```python
class Singleton:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.init_singleton()
        return cls._instance
    
    def init_singleton(self):
        self.value = "Инициализировано"
    
    def business_logic(self):
        print(f"Выполнена логика. Состояние: {self.value}")

# Клиентский код
s1 = Singleton()
s2 = Singleton()

s1.value = "Изменено"
print(s1 is s2)  # True
s2.business_logic()  # Выполнена логика. Состояние: Изменено
```

**Потокобезопасная версия:**
```python
import threading

class ThreadSafeSingleton:
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls):
        with cls._lock:
            if cls._instance is None:
                cls._instance = super().__new__(cls)
        return cls._instance
```

---

# Пример На C#
**Базовая реализация:**
```csharp
public sealed class Singleton
{
    private static Singleton _instance;
    
    private Singleton() 
    {
        // Приватный конструктор
        Value = "Инициализировано";
    }
    
    public static Singleton Instance
    {
        get
        {
            _instance ??= new Singleton();
            return _instance;
        }
    }
    
    public string Value { get; set; }
    
    public void BusinessLogic()
    {
        Console.WriteLine($"Выполнена логика. Состояние: {Value}");
    }
}

// Клиентский код
var s1 = Singleton.Instance;
var s2 = Singleton.Instance;

s1.Value = "Изменено";
Console.WriteLine(s1 == s2);  // True
s2.BusinessLogic();  // Выполнена логика. Состояние: Изменено
```

**Потокобезопасная версия (Lazy):**
```csharp
public sealed class ThreadSafeSingleton
{
    private static readonly Lazy<ThreadSafeSingleton> _lazy = 
        new Lazy<ThreadSafeSingleton>(() => new ThreadSafeSingleton());
    
    public static ThreadSafeSingleton Instance => _lazy.Value;
    
    private ThreadSafeSingleton() { }
}
```

---

# Против Каких Антипаттернов Борется Singleton?
1. **Глобальные переменные (Global State):**
   **Проблема:** Неуправляемые глобальные переменные создают хаотичные зависимости.
   **Решение:** Singleton инкапсулирует состояние и контролирует доступ.

2. **Бесконтрольное создание объектов:**
   **Проблема:** Множественные экземпляры класса, который должен быть единственным (например, подключение к БД).
   **Решение:** Гарантирует единственность экземпляра.

3. **Расточительное использование ресурсов:**
   **Проблема:** Повторное создание ресурсоёмких объектов (кэш, пулы соединений).
   **Решение:** Единый экземпляр переиспользуется.

4. **Нарушение [[SOLID принципы#^eb6306|SRP (Single Responsibility Principle)]]:**
   **Проблема:** Класс берет на себя управление своим жизненным циклом.
   **Решение:** Делегирование создания статическому фабричному методу.

---

# Когда Использовать Singleton?
1. Для ресурсов, существующих в единственном экземпляре:
   - Логгеры
   - Кэши
   - Подключения к БД
   - Конфигурация приложения
2. Когда необходим строгий контроль глобального состояния
3. Для замены глобальных переменных

---

# Опасности И Ограничения:
1. **Нарушение модульности:** Скрытые зависимости через глобальный доступ
2. **Сложность тестирования:** Глобальное состояние сохраняется между тестами
3. **Антипаттерн при неправильном использовании:**
   - Использование для объектов, которые не должны быть единственными
   - Чрезмерное применение в многослойной архитектуре
4. **Потокобезопасность:** Требует особой реализации в многопоточных средах

> **Важно:** В современных приложениях предпочтительно использовать [[Dependency Injection основы и применение|Dependency Injection]] вместо прямого доступа к Singleton, так как это упрощает тестирование и соблюдает принцип инверсии зависимостей.