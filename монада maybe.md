---
tags:
  - csharp
---

В C# монады можно реализовать с помощью оберток для значений и методов, которые позволяют работать с ними последовательно. Один из самых простых примеров монады — это Maybe (или Option), которая используется для обработки значений, которые могут быть отсутствующими (null).
Вот пример реализации монады Maybe в C#:
```cs
using System;

public class Maybe<T>
{
    private readonly T _value;
    public bool HasValue { get; }

    private Maybe() { HasValue = false; }
    
    public Maybe(T value)
    {
        _value = value;
        HasValue = true;
    }

    public static Maybe<T> Nothing() => new Maybe<T>();

    public Maybe<TResult> Bind<TResult>(Func<T, Maybe<TResult>> func)
    {
        if (HasValue)
        {
            return func(_value);
        }
        return Maybe<TResult>.Nothing();
    }

    public TResult GetValueOrDefault(TResult defaultValue)
    {
        return HasValue ? (TResult)(object)_value : defaultValue;
    }
}

class Program
{
    static void Main()
    {
        var maybeValue = new Maybe<int>(5);

        var result = maybeValue
            .Bind(x => new Maybe<int>(x * 2)) // Умножаем на 2
            .Bind(x => x > 5 ? new Maybe<int>(x) : Maybe<int>.Nothing()); // Проверяем, больше ли 5

        Console.WriteLine(result.GetValueOrDefault(-1)); // Вывод: 10

        var noneValue = Maybe<int>.Nothing();

        var noneResult = noneValue
            .Bind(x => new Maybe<int>(x * 2))
            .Bind(x => x > 5 ? new Maybe<int>(x) : Maybe<int>.Nothing());
        Console.WriteLine(noneResult.GetValueOrDefault(-1)); // Вывод: -1
    }
}
```
▎Объяснение:
1. Класс `Maybe<T>`:
   • Содержит значение типа T и флаг HasValue, который указывает, есть ли значение.
   • Конструктор принимает значение и устанавливает флаг HasValue в true.
2. Метод Bind:
   • Принимает функцию, которая возвращает `Maybe<TResult>`.
   • Если значение присутствует, вызывает функцию и возвращает результат; если нет — возвращает Nothing.
3. Метод `GetValueOrDefault`:
   • Возвращает значение, если оно присутствует, или значение по умолчанию.
4. Пример использования:
   • Создаем `Maybe<int>` с значением 5.
   • Применяем последовательные операции через Bind.
   • Выводим результат, который будет 10.
   • Проверяем случай с отсутствующим значением, где результат будет -1.
Этот пример демонстрирует, как монада может помочь в управлении значениями, которые могут отсутствовать, и упрощает обработку таких случаев.