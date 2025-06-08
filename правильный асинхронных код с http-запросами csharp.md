---
tags:
  - csharp
  - JSON
---

Вот как правильно работать с асинхронным кодом и HTTP-запросами в C#:

```csharp
// 1. Базовый асинхронный запрос с HttpClient
public async Task<string> GetDataAsync()
{
    using var client = new HttpClient();
    try
    {
        var response = await client.GetAsync("https://api.example.com/data");
        response.EnsureSuccessStatusCode(); // Проверка кода ответа
        return await response.Content.ReadAsStringAsync();
    }
    catch (HttpRequestException ex)
    {
        Console.WriteLine($"Ошибка запроса: {ex.Message}");
        throw;
    }
}

// 2. Обработка ошибок с try-catch
public async Task<Data> GetJsonDataAsync()
{
    using var client = new HttpClient();
    try
    {
        var response = await client.GetAsync("https://api.example.com/data");
        response.EnsureSuccessStatusCode();
        
        return await response.Content.ReadFromJsonAsync<Data>();
    }
    catch (HttpRequestException ex)
    {
        Console.WriteLine($"HTTP Error: {ex.StatusCode} - {ex.Message}");
        return default;
    }
    catch (JsonException ex)
    {
        Console.WriteLine($"JSON Parsing Error: {ex.Message}");
        throw;
    }
}

// 3. Параллельные запросы с Task.WhenAll
public async Task<(Data data1, Data data2)> GetMultipleDataAsync()
{
    using var client = new HttpClient();
    
    var task1 = client.GetFromJsonAsync<Data>("https://api.example.com/data1");
    var task2 = client.GetFromJsonAsync<Data>("https://api.example.com/data2");
    
    var results = await Task.WhenAll(task1, task2);
    return (results[0], results[1]);
}

// 4. Последовательные запросы
public async Task<CombinedResult> GetSequentialDataAsync()
{
    using var client = new HttpClient();
    
    var user = await client.GetFromJsonAsync<User>("https://api.example.com/user");
    var posts = await client.GetFromJsonAsync<List<Post>>($"https://api.example.com/posts/{user.Id}");
    
    return new CombinedResult { User = user, Posts = posts };
}

// 5. POST-запрос с JSON телом
public async Task<Response> PostDataAsync(Request request)
{
    using var client = new HttpClient();
    var content = new StringContent(
        JsonSerializer.Serialize(request),
        Encoding.UTF8,
        "application/json");
    
    var response = await client.PostAsync("https://api.example.com/data", content);
    response.EnsureSuccessStatusCode();
    
    return await response.Content.ReadFromJsonAsync<Response>();
}

// 6. Работа с CancellationToken
public async Task<Data> GetDataWithCancellationAsync(CancellationToken ct)
{
    using var client = new HttpClient();
    try
    {
        var response = await client.GetAsync("https://api.example.com/data", ct);
        response.EnsureSuccessStatusCode();
        return await response.Content.ReadFromJsonAsync<Data>(cancellationToken: ct);
    }
    catch (TaskCanceledException)
    {
        Console.WriteLine("Запрос отменен");
        return default;
    }
}

// DTO-классы для примера
public class Data { /* свойства */ }
public class User { public int Id { get; set; } }
public class Post { /* свойства */ }
public class CombinedResult { public User User; public List<Post> Posts; }

// Пример использования в приложении
public async Task Main()
{
    try
    {
        var data = await GetJsonDataAsync();
        Console.WriteLine($"Получено данных: {data}");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Критическая ошибка: {ex}");
    }
}
```

**Рекомендации для C#:**

1. **Используйте `async`/`await`** вместо `.Result` или `.Wait()` для избежания deadlock
2. **Правильное управление HttpClient**:
   - Используйте IHttpClientFactory в ASP.NET Core
   - Или создавайте один экземпляр HttpClient и переиспользуйте его
3. **Обработка ошибок**:
   - Проверяйте статус код через `EnsureSuccessStatusCode()`
   - Ловите специфические исключения (HttpRequestException, JsonException)
4. **Отмена операций**:
   - Всегда передавайте CancellationToken в асинхронные методы
5. **Сериализация/Десериализация**:
   - Используйте `System.Text.Json` для работы с JSON
   - Настройте политики сериализации при необходимости
6. **Конфигурация запросов**:
   - Устанавливайте Timeout для HttpClient
   - Добавляйте заголовки через `DefaultRequestHeaders`

**Пример с IHttpClientFactory (ASP.NET Core):**

```csharp
public class DataService
{
    private readonly HttpClient _client;
    
    public DataService(IHttpClientFactory clientFactory)
    {
        _client = clientFactory.CreateClient("ExampleClient");
    }
    
    public async Task<Data> GetDataAsync()
    {
        var response = await _client.GetAsync("/data");
        response.EnsureSuccessStatusCode();
        return await response.Content.ReadFromJsonAsync<Data>();
    }
}

// В Startup.cs
services.AddHttpClient("ExampleClient", client => 
{
    client.BaseAddress = new Uri("https://api.example.com");
    client.Timeout = TimeSpan.FromSeconds(30);
});
```

**Лучшие практики:**
- Всегда используйте `ConfigureAwait(false)` в библиотечных проектах
- Избегайте `async void` (кроме event handlers)
- Используйте ValueTask<T> для методов, которые могут завершаться синхронно
- Логируйте ошибки и отслеживайте тайм-ауты
- Используйте Polly для реализации retry-логики

Эти подходы помогут создавать эффективный и надежный асинхронный код в C# с правильной обработкой HTTP-запросов и ошибок.