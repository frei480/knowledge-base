---
tags:
  - csharp
  - JSON
---

Разберу последний пример подробно с объяснением каждой части и нюансов работы:

```csharp
using System.Net.Http.Json;

public static async Task<T?> GetApiResponse<T>(string url)
{
    using var client = new HttpClient();
    return await client.GetFromJsonAsync<T>(url);
}
```

### 1. Подключение зависимостей
```csharp
using System.Net.Http.Json;
```
- Требует подключения NuGet пакета `System.Net.Http.Json`
- Доступен в .NET 5 и новее

---

### 2. Сигнатура метода
```csharp
public static async Task<T?> GetApiResponse<T>(string url)
```
- `T?` – nullable возвращаемый тип (допускает значение null)
- `async Task` – асинхронная операция
- Обобщенный тип `<T>` – позволяет работать с любым типом данных

---

### 3. Использование HttpClient
```csharp
using var client = new HttpClient();
```
- `using` гарантирует корректное освобождение ресурсов
- В реальных приложениях лучше использовать `IHttpClientFactory`
- Создание нового HttpClient для каждого вызова – антипаттерн для продакшена

---

### 4. Основная операция
```csharp
return await client.GetFromJsonAsync<T>(url);
```
Метод `GetFromJsonAsync` выполняет сразу три операции:
1. Отправляет GET-запрос по указанному URL
2. Проверяет статус ответа (выбрасывает исключение при ошибке HTTP)
3. Десериализует JSON-ответ в указанный тип `<T>`

---

### Пример модели данных
```csharp
public class WeatherForecast
{
    public DateTime Date { get; set; }
    public int TemperatureC { get; set; }
    public string? Summary { get; set; }
}
```

Соответствует JSON:
```json
{
    "date": "2023-08-20T12:00:00",
    "temperatureC": 25,
    "summary": "Warm"
}
```

---

### Полный пример использования с обработкой ошибок
```csharp
try
{
    var forecast = await GetApiResponse<WeatherForecast>(
        "https://api.weather.com/forecast/today"
    );
    
    if (forecast != null)
    {
        Console.WriteLine($"Температура: {forecast.TemperatureC}°C");
    }
}
catch (HttpRequestException ex)
{
    Console.WriteLine($"Ошибка HTTP: {ex.StatusCode} - {ex.Message}");
}
catch (JsonException ex)
{
    Console.WriteLine($"Ошибка парсинга JSON: {ex.Message}");
}
```

---

### Важные нюансы:
1. **Nullability**
- Возвращает `T?` (может быть null), если:
  - Сервер вернул 204 No Content
  - Ответ имеет неожиданную структуру
  - Произошла ошибка десериализации

2. **Исключения:**
- `HttpRequestException` – при ошибках HTTP (404, 500 и т.д.)
- `JsonException` – при проблемах с десериализацией
- `TaskCanceledException` – при таймауте запроса

3. **Настройка десериализации:**
Для кастомных настроек можно передать `JsonSerializerOptions`:
```csharp
await client.GetFromJsonAsync<T>(
    url, 
    new JsonSerializerOptions 
    {
        PropertyNameCaseInsensitive = true
    }
);
```

4. **Производительность:**
- Для частых запросов используйте один экземпляр HttpClient
- Добавьте политику повторов (Polly)
- Настройте таймауты:
```csharp
client.Timeout = TimeSpan.FromSeconds(30);
```

---

### Альтернативная реализация с улучшениями
```csharp
public static async Task<T?> GetApiResponse<T>(
    string url, 
    JsonSerializerOptions? options = null,
    CancellationToken ct = default
)
{
    using var client = new HttpClient();
    client.Timeout = TimeSpan.FromSeconds(15);
    
    var response = await client.GetAsync(url, ct);
    response.EnsureSuccessStatusCode();
    
    return await response.Content.ReadFromJsonAsync<T>(
        options, 
        cancellationToken: ct
    );
}
```

Особенности этой версии:
- Поддержка cancellation token
- Кастомные настройки сериализации
- Явная проверка статуса кода
- Настройка таймаута

Этот подход лучше подходит для продакшн-окружения, сохраняя простоту использования.