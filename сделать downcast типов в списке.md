---
tags:
  - csharp
---

Downcast (или "снижающее приведение") типов в C# — это процесс приведения объекта к более конкретному типу, который он фактически представляет. При этом важно помнить, что downcast может привести к исключению InvalidCastException, если объект не может быть приведен к указанному типу.

Для демонстрации downcast типов в списке, давайте создадим пример, в котором у нас есть базовый класс и несколько производных классов. Мы будем хранить объекты этих классов в списке базового типа и затем выполнять downcast для доступа к специфическим свойствам производных классов.

▎Пример
```cs
using System;
using System.Collections.Generic;

class Animal
{
    public string Name { get; set; }
}

class Dog : Animal
{
    public void Bark()
    {
        Console.WriteLine($"{Name} says: Woof!");
    }
}

class Cat : Animal
{
    public void Meow()
    {
        Console.WriteLine($"{Name} says: Meow!");
    }
}

class Program
{
    static void Main()
    {
        // Создаем список животных
        List<Animal> animals = new List<Animal>
        {
            new Dog { Name = "Buddy" },
            new Cat { Name = "Whiskers" },
            new Dog { Name = "Max" }
        };

        // Проходим по списку и выполняем downcast
        foreach (var animal in animals)
        {
            if (animal is Dog dog)
            {
                dog.Bark(); // Вызываем метод Bark для собак
            }
            else if (animal is Cat cat)
            {
                cat.Meow(); // Вызываем метод Meow для кошек
            }
        }
    }
}

```
▎Объяснение кода:
1. Создание базового и производных классов:
   • Animal — базовый класс с одним свойством Name.
   • Dog и Cat — производные классы с методами Bark() и Meow() соответственно.
2. Создание списка: Мы создаем список animals, который содержит объекты типа Dog и Cat, но все они хранятся как объекты типа Animal.
3. Downcast с использованием оператора is: 
   • Мы проходим по каждому элементу списка. 
   • С помощью оператора is мы проверяем, является ли текущий элемент объектом типа Dog. Если да, то выполняем downcast и вызываем метод Bark().
   • Аналогично, если элемент является объектом типа Cat, мы вызываем метод Meow().

▎Результат выполнения программы:
```
Buddy says: Woof!
Whiskers says: Meow!
Max says: Woof!
```
Таким образом, мы успешно выполнили downcast типов из списка, чтобы вызывать методы, специфичные для производных классов.