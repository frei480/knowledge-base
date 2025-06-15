---
tags:
  - АрхитектураПО
---
# Паттерн Chain of Responsibility (Цепочка обязанностей)
**Назначение:**
Позволяет передавать запросы последовательно по цепочке обработчиков. Каждый обработчик решает, обработать запрос или передать следующему в цепи. Устраняет жёсткую связь между отправителем и получателем.

**Ключевые компоненты:**
1. **Handler** - интерфейс обработчика (метод `handle_request`)
2. **ConcreteHandler** - конкретные обработчики (реализуют логику обработки и передачу дальше)
3. **Client** - инициирует запрос к первому обработчику в цепи

---

# Пример На Python
```python
from abc import ABC, abstractmethod
from typing import Optional

class Handler(ABC):
    @abstractmethod
    def set_next(self, handler):
        pass

    @abstractmethod
    def handle(self, request: int) -> Optional[str]:
        pass

class BaseHandler(Handler):
    _next: Handler = None

    def set_next(self, handler: Handler):
        self._next = handler
        return handler  # Для построения цепочки в стиле fluent

    def handle(self, request: int):
        if self._next:
            return self._next.handle(request)
        return None

class NegativeHandler(BaseHandler):
    def handle(self, request: int):
        if request < 0:
            return f"NegativeHandler: обработал {request}"
        return super().handle(request)

class ZeroHandler(BaseHandler):
    def handle(self, request: int):
        if request == 0:
            return f"ZeroHandler: обработал {request}"
        return super().handle(request)

class PositiveHandler(BaseHandler):
    def handle(self, request: int):
        if request > 0:
            return f"PositiveHandler: обработал {request}"
        return super().handle(request)

# Клиентский код
chain = NegativeHandler()
chain.set_next(ZeroHandler()).set_next(PositiveHandler())

requests = [-5, 0, 7]
for req in requests:
    result = chain.handle(req)
    print(f"Запрос {req}: {result or 'не обработан'}")
```

**Вывод:**
```
Запрос -5: NegativeHandler: обработал -5
Запрос 0: ZeroHandler: обработал 0
Запрос 7: PositiveHandler: обработал 7
```

---

# Пример На C#
```csharp
using System;

public interface IHandler
{
    IHandler SetNext(IHandler handler);
    object Handle(int request);
}

public abstract class AbstractHandler : IHandler
{
    private IHandler _next;

    public IHandler SetNext(IHandler handler)
    {
        _next = handler;
        return handler;
    }

    public virtual object Handle(int request)
    {
        return _next?.Handle(request);
    }
}

public class NegativeHandler : AbstractHandler
{
    public override object Handle(int request)
    {
        if (request < 0)
            return $"NegativeHandler: обработал {request}";
        return base.Handle(request);
    }
}

public class ZeroHandler : AbstractHandler
{
    public override object Handle(int request)
    {
        if (request == 0)
            return $"ZeroHandler: обработал {request}";
        return base.Handle(request);
    }
}

public class PositiveHandler : AbstractHandler
{
    public override object Handle(int request)
    {
        if (request > 0)
            return $"PositiveHandler: обработал {request}";
        return base.Handle(request);
    }
}

class Program
{
    static void Main()
    {
        var chain = new NegativeHandler();
        chain.SetNext(new ZeroHandler())
             .SetNext(new PositiveHandler());

        int[] requests = { -5, 0, 7 };
        foreach (var req in requests)
        {
            var result = chain.Handle(req) ?? "не обработан";
            Console.WriteLine($"Запрос {req}: {result}");
        }
    }
}
```

**Вывод:**
```
Запрос -5: NegativeHandler: обработал -5
Запрос 0: ZeroHandler: обработал 0
Запрос 7: PositiveHandler: обработал 7
```

---

# Против Каких Антипаттернов Применяется?
1. **God Object (Божественный объект)**
   Заменяет монолитный класс, делающий «всё», на несколько специализированных обработчиков.

2. **Spaghetti Code (Спагетти-код)**
   Устраняет длинные цепочки условных операторов (`if/else` или `switch`), заменяя их цепочкой объектов.

3. **Feature Envy (Зависть к функционалу)**
   Избавляет классы от необходимости знать о логике обработки других классов, делегируя запросы по цепочке.

4. **Tight Coupling (Жёсткая связанность)**
   Ослабляет связи между отправителем запроса и его получателями.

---

# Когда Использовать?
- Есть несколько объектов, способных обработать запрос
- Обработчики должны определяться динамически
- Необходима последовательная обработка запроса разными модулями
- Клиент не должен знать, какой именно обработчик выполнит задачу

**Плюсы:**
- Гибкость в распределении обязанностей
- Уменьшение зависимости между компонентами
- Возможность динамического изменения цепочки

**Минусы:**
- Нет гарантии обработки запроса (может «пройти» всю цепь)
- Сложность отладки из-за распределённого поведения