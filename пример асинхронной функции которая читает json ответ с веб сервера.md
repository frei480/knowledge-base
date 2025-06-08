---
tags:
  - csharp
  - JSON
---

Вот пример асинхронной функции на C#, которая читает JSON-ответ с веб-сервера с использованием `HttpClient` и `System.Text.Json`:

```csharp
using System;
using System.Net.Http;
using System.Net.Http.Json;
using System.Threading.Tasks;

public class ApiService
{
    private readonly HttpClient _httpClient;

    // Рекомендуется использовать HttpClient через фабрику (IHttpClientFactory) в реальных приложениях
    public ApiService(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    // Обобщенный метод для получения и десериализации JSON
    public async Task<T> GetJsonResponseAsync<T>(string endpoint)
    {
        try
        {
            var response = await _httpClient.GetAsync(endpoint);
            response.EnsureSuccessStatusCode(); // Проверка на успешный статус
            return await response.Content.ReadFromJsonAsync<T>();
        }
        catch (HttpRequestException ex)
        {
            Console.WriteLine($"Ошибка HTTP: {ex.Message}");
            throw;
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Ошибка: {ex.Message}");
            throw;
        }
    }
}

// Пример использования
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}

public class Program
{
    public static async Task Main()
    {
        using var httpClient = new HttpClient();
        var apiService = new ApiService(httpClient);

        try
        {
            var user = await apiService.GetJsonResponseAsync<User>("https://api.example.com/users/1");
            Console.WriteLine($"User: {user.Name}, Email: {user.Email}");
        }
        catch
        {
            Console.WriteLine("Не удалось получить данные");
        }
    }
}
```

Ключевые особенности:
1. Использует `HttpClient` для асинхронных HTTP-запросов
2. Десериализует JSON-ответ сразу в указанный тип (`T`)
3. Обрабатывает ошибки HTTP и исключения
4. Следует рекомендациям:
   - Инъекция зависимостей для `HttpClient`
   - Использование `EnsureSuccessStatusCode()`
   - Асинхронные методы с `async`/`await`
   - Обобщенный возвращаемый тип

Для работы с JSON необходимо:
1. Добавить NuGet-пакет `System.Net.Http.Json`
2. Определить классы-модели, соответствующие структуре JSON-ответа

Пример для .NET 6+ с использованием новых возможностей:

```csharp
using System.Net.Http.Json;

public static async Task<T?> GetApiResponse<T>(string url)
{
    using var client = new HttpClient();
    return await client.GetFromJsonAsync<T>(url);
}
```

Для использования этого метода:[[пример с использованием GetFromJsonAsync NET 6]]
```csharp
var data = await GetApiResponse<MyDataModel>("https://api.example.com/data");
```
[[если json содержит список объектов]]