---
tags:
  - csharp
---

Для использования метода `Distinct` с кастомным компаратором в C# необходимо реализовать интерфейс `IEqualityComparer<T>`. Этот интерфейс требует определения двух методов: `Equals` и `GetHashCode`. Вот как это сделать:

**1. Реализация интерфейса `IEqualityComparer<T>`**

Пример для сравнения объектов `Person` по свойству `Id`:

```csharp
public class Person
{
    public int Id { get; set; }
    public string Name { get; set; }
}

public class PersonComparer : IEqualityComparer<Person>
{
    public bool Equals(Person x, Person y)
    {
        if (ReferenceEquals(x, y)) return true;
        if (x is null || y is null) return false;
        return x.Id == y.Id;
    }

    public int GetHashCode(Person obj)
    {
        return obj.Id.GetHashCode();
    }
}
```

**2. Использование компаратора в методе Distinct**

```csharp
List<Person> people = new List<Person>
{
    new Person { Id = 1, Name = "Alice" },
    new Person { Id = 1, Name = "Alice" },
    new Person { Id = 2, Name = "Bob" }
};

var distinctPeople = people.Distinct(new PersonComparer());
```

**Результат:** В коллекции останутся объекты с Id 1 и 2.

**Важные нюансы:**
- Всегда корректно реализуйте `GetHashCode`. Для равных объектов хеш-коды должны совпадать.
- Обрабатывайте `null` значения в методе `Equals`.
- Стандартный `Distinct` не принимает лямбды, но можно использовать пакет [MoreLINQ](https://github.com/morelinq/MoreLINQ) или создать метод-обёртку.

**Альтернатива без компаратора (через LINQ):**
```csharp
var distinctById = people.GroupBy(p => p.Id).Select(g => g.First());
```