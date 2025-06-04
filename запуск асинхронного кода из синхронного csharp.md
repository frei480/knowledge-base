---
tags:
  - csharp
---
В C# вы можете запускать асинхронный код из синхронного с помощью метода `GetAwaiter().GetResult()` или `Task.Result`, но важно помнить, что это может привести к блокировке потока, если асинхронный метод использует синхронизацию контекста (например, в приложениях с пользовательским интерфейсом).

Вот пример, как можно запустить асинхронный метод из синхронного:
```cs
using System;
using System.Threading.Tasks;

class Program
{
    static void Main(string[] args)
    {
        // Запускаем асинхронный метод из синхронного
        var result = RunAsyncMethod().GetAwaiter().GetResult();
        Console.WriteLine(result);
    }

    static async Task<string> RunAsyncMethod()
    {
        await Task.Delay(1000); // Имитация асинхронной работы
        return "Асинхронная операция завершена!";
    }
}
```
▎Варианты запуска

1. Использование GetAwaiter().GetResult():
   Этот способ позволяет избежать некоторых проблем с блокировкой потока, но все же может привести к Deadlock в контексте UI-приложений.

2. Использование Task.Run:
   Если вы хотите избежать блокировки текущего потока, вы можете использовать Task.Run для запуска асинхронного метода:

```cs
   static void Main(string[] args)
   {
       var result = Task.Run(async () => await RunAsyncMethod()).GetAwaiter().GetResult();
       Console.WriteLine(result);
   }
```
   
3. Использование async Main (начиная с C# 7.1):
   Если ваша версия C# поддерживает async Main, вы можете сделать ваш метод Main асинхронным:
   
```cs
   using System;
   using System.Threading.Tasks;

   class Program
   {
       static async Task Main(string[] args)
       {
           var result = await RunAsyncMethod();
           Console.WriteLine(result);
       }

       static async Task<string> RunAsyncMethod()
       {
           await Task.Delay(1000);
           return "Асинхронная операция завершена!";
       }
   }
```
   

Выбор метода зависит от вашего конкретного случая использования и контекста, в котором вы работаете.