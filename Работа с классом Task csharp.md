---
tags:
  - csharp
---
Работа с классом Task
Вложенные задачи
Одна задача может запускать другую - вложенную задачу. При этом эти задачи выполняются независимо друг от друга. Например:


```cs
var outer = Task.Factory.StartNew(() => // внешняя задача
{
    Console.WriteLine("Outer task starting...");
    var inner = Task.Factory.StartNew(() => // вложенная задача
    {
        Console.WriteLine("Inner task starting...");
        Thread.Sleep(2000);
        Console.WriteLine("Inner task finished.");
    });
});

outer.Wait(); // ожидаем выполнения внешней задачи
Console.WriteLine("End of Main");
```

Несмотря на то, что здесь мы ожидаем выполнения внешней задачи, но вложенная задача может завершить выполнение даже после завершения метода Main:
```
Outer task starting...
End of Main
```

При этом внутренняя задача может даже не начать свое выполнение к завершению работы основного потока программы. То есть в данном случае внешняя и вложенная задачи выполняются независимо друг от друга.
Если необходимо, чтобы вложенная задача выполнялась как часть внешней, необходимо использовать значение TaskCreationOptions.AttachedToParent:
```cs

var outer = Task.Factory.StartNew(() => // внешняя задача

{

    Console.WriteLine("Outer task starting...");

    var inner = Task.Factory.StartNew(() => // вложенная задача

    {

        Console.WriteLine("Inner task starting...");

        Thread.Sleep(2000);

        Console.WriteLine("Inner task finished.");

    }, TaskCreationOptions.AttachedToParent);

});

outer.Wait(); // ожидаем выполнения внешней задачи

Console.WriteLine("End of Main");
```

Консольный вывод:

```
Outer task starting...
Inner task starting...
Inner task finished.
End of Main
```

В данном случае вложенная задача прикреплена к внешней и выполняется как часть внешней задачи. И внешняя задача завершится только когда завершатся все прикрепленные к ней вложенные задачи.



### Массив задач

Также как и с потоками, мы можем создать и запустить массив задач. Можно определить все задачи в массиве непосредственно через объект Task:


```cs
Task[] tasks1 = new Task[3]

{

    new Task(() => Console.WriteLine("First Task")),

    new Task(() => Console.WriteLine("Second Task")),

    new Task(() => Console.WriteLine("Third Task"))

};

// запуск задач в массиве

foreach (var t in tasks1)

    t.Start();
```

Либо также можно использовать методы Task.Factory.StartNew или Task.Run и сразу запускать все задачи:

```cs

Task[] tasks2 = new Task[3];

int j = 1;

for (int i = 0; i < tasks2.Length; i++)

    tasks2[i] = Task.Factory.StartNew(() => Console.WriteLine($"Task {j++}"));
```
Но в любом случае мы опять же можем столкнуться с тем, что все задачи из массива могут завершиться после того, как отработает метод Main, в котором запускаются эти задачи:

```cs

Task[] tasks = new Task[3];

for(var i = 0; i < tasks.Length; i++)

{

    tasks[i] = new Task(() =>

    {

        Thread.Sleep(1000); // эмуляция долгой работы

        Console.WriteLine($"Task{i} finished");

    });

    tasks[i].Start(); // запускаем задачу

}

Console.WriteLine("Завершение метода Main");
```
Один из возможных консольных выводов программы:
```
Завершение метода Main
```

Если необходимо завершить выполнение программы или вообще выполнять некоторый код лишь после того, как все задачи из массива завершатся, то применяется метод Task.WaitAll(tasks):

```cs

Task[] tasks = new Task[3];

for(var i = 0; i < tasks.Length; i++)

{

    tasks[i] = new Task(() =>

    {

        Thread.Sleep(1000); // эмуляция долгой работы

        Console.WriteLine($"Task{i} finished");

    });

    tasks[i].Start(); // запускаем задачу

}

Console.WriteLine("Завершение метода Main");

Task.WaitAll(tasks); // ожидаем завершения всех задач
```
В этом случае сначала завершатся все задачи, и лишь только потом будет выполняться последующий код из метода Main:


```
Завершение метода Main

Task3 finished

Task3 finished

Task3 finished
```

В то же время порядок выполнения самих задач в массиве также недетерминирован.



Также мы можем применять метод Task.WaitAny(tasks). Он ждет, пока завершится хотя бы одна из массива задач.



Возвращение результатов из задач

Задачи могут не только выполняться как процедуры, но и возвращать определенные результаты:

```cs

int n1 =4, n2 = 5;

Task<int> sumTask = new Task<int>(() => Sum(n1, n2));

sumTask.Start();

int result = sumTask.Result;

Console.WriteLine($"{n1} + {n2} = {result}"); // 4 + 5 = 9

int Sum(int a, int b) => a + b;
```

Во-первых, чтобы получать из задачи не который результат, необходимо типизировать объект Task тем типом, объект которого мы хотим получить из задачи. Например, в примере выше мы ожидаем из задачи sumTask получить число типа int, соответственно типизируем объект Task данным типом - `Task<int>`.



И, во-вторых, в качестве задачи должен выполняться метод, который возвращает данный тип объекта. Так, в данном случае у нас в качестве задачи выполняется метод Sum, которая принимает два числа и на выходе возвращает их сумму - значение типа int.



Возвращаемое число будет храниться в свойстве Result: sumTask.Result. Нам не надо его приводить к типу int, оно уже само по себе будет представлять число.



int result = sumTask.Result;

При этом при обращении к свойству Result текущий поток останавливает выполнение и ждет, когда будет получен результат из выполняемой задачи.



Другой пример:

```cs

Task<Person> defaultPersonTask = new Task<Person>(() => new Person("Tom", 37));

defaultPersonTask.Start();

Person defaultPerson = defaultPersonTask.Result;

Console.WriteLine($"{defaultPerson.Name} - {defaultPerson.Age}"); // Tom - 37

record class Person(string Name, int Age); 
```

В данном случае задача defaultPersonTask возвращает объект типа Person, который мы можем получить из свойства Result.