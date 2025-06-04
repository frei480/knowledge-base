---
tags:
  - csharp
---

Если JSON содержит массив (список) объектов, подход очень похож на работу с одиночным объектом, но с указанием соответствующего типа коллекции. Разберём решение подробно.

---

### 1. Модель данных
Допустим, сервер возвращает JSON вида:
```json
[
    {
        "id": 1,
        "name": "Alice",
        "email": "alice@example.com"
    },
    {
        "id": 2,
        "name": "Bob",
        "email": "bob@example.com"
    }
]
```

Определяем класс для элемента списка:
```csharp
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}
```

---

### 2. Пример асинхронного метода
Модифицируем предыдущий метод, указав тип `List<User>` (или `IEnumerable<User>`, `User[]`):

```csharp
using System.Collections.Generic;
using System.Net.Http;
using System.Net.Http.Json;
using System.Threading.Tasks;

public static async Task<List<User>> GetUsersAsync(string url)
{
    using var client = new HttpClient();
    return await client.GetFromJsonAsync<List<User>>(url) 
        ?? new List<User>(); // Возвращаем пустой список при null
}
```

**Или обобщённая версия:**
```csharp
public static async Task<List<T>> GetListAsync<T>(string url)
{
    using var client = new HttpClient();
    var result = await client.GetFromJsonAsync<List<T>>(url);
    return result ?? new List<T>();
}
```

---

### 3. Использование
```csharp
try
{
    var users = await GetUsersAsync("https://api.example.com/users");
    
    foreach (var user in users)
    {
        Console.WriteLine($"User: {user.Name}, ID: {user.Id}");
    }
}
catch (HttpRequestException ex)
{
    Console.WriteLine($"HTTP Error: {ex.StatusCode}");
}
```

---

### 4. Особенности работы с коллекциями

1. **Типы коллекций:**
   ```csharp
   // Все варианты корректны
   GetFromJsonAsync<User[]>(url)
   GetFromJsonAsync<IEnumerable<User>>(url)
   GetFromJsonAsync<List<User>>(url)
   ```

2. **Обработка пустого ответа:**
   - Если сервер вернёт `204 No Content`, метод вернёт `null`
   - Используйте оператор `??` для подстраховки:
     ```csharp
     var users = await GetFromJsonAsync<List<User>>(url) ?? new();
     ```

3. **Кастомная десериализация:**
   Если имена свойств в JSON не совпадают с моделью:
   ```csharp
   var options = new JsonSerializerOptions
   {
       PropertyNameCaseInsensitive = true
   };

   var users = await client.GetFromJsonAsync<List<User>>(url, options);
   ```

---

### 5. Полный пример с обработкой ошибок
```csharp
public static async Task Main()
{
    try
    {
        var users = await GetListAsync<User>("https://api.example.com/users");
        
        if (users.Count == 0)
        {
            Console.WriteLine("Список пользователей пуст");
            return;
        }

        foreach (var user in users)
        {
            Console.WriteLine($"{user.Id}: {user.Name}");
        }
    }
    catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.NotFound)
    {
        Console.WriteLine("Ресурс не найден");
    }
    catch (JsonException ex)
    {
        Console.WriteLine($"Ошибка формата данных: {ex.Message}");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Неизвестная ошибка: {ex.Message}");
    }
}
```

---

### 6. Важные нюансы

1. **Производительность:** Для больших коллекций используйте:
   ```csharp
   await response.Content.ReadFromJsonAsAsyncEnumerable<T>();
   ```
   Это позволяет обрабатывать элементы по мере их получения.

2. **Пагинация:** Если API возвращает данные с пагинацией:
   ```json
   {
       "page": 1,
       "totalPages": 5,
       "items": [...]
   }
   ```
   Создайте модель-обёртку:
   ```csharp
   public class PagedResponse<T>
   {
       public int Page { get; set; }
       public int TotalPages { get; set; }
       public List<T> Items { get; set; }
   }
   ```
   Использование:
   ```csharp
   var response = await client.GetFromJsonAsync<PagedResponse<User>>(url);
   var users = response.Items;
   ```

3. **Сериализация полей:** Если JSON содержит вложенные массивы:
   ```json
   {
       "users": [...],
       "metadata": {...}
   }
   ```
   Используйте соответствующие свойства в модели:
   ```csharp
   public class ApiResponse
   {
       public List<User> Users { get; set; }
       public Metadata Meta { get; set; }
   }
   ```

---

### 7. Альтернатива: ручная десериализация
Если нужен полный контроль над процессом:
```csharp
using var response = await client.GetAsync(url);
var jsonString = await response.Content.ReadAsStringAsync();
var users = JsonSerializer.Deserialize<List<User>>(jsonString, options);
```

Этот подход полезен при:
- Нестандартных структурах JSON
- Необходимости предобработки JSON
- Работе с legacy-системами

---

### 8. Тестирование
Пример теста с Moq:
```csharp
[Test]
public async Task GetUsers_ReturnsValidList()
{
    // Arrange
    var mockHandler = new Mock<HttpMessageHandler>();
    mockHandler.Protected()
        .Setup<Task<HttpResponseMessage>>(
            "SendAsync",
            ItExpr.IsAny<HttpRequestMessage>(),
            ItExpr.IsAny<CancellationToken>()
        )
        .ReturnsAsync(new HttpResponseMessage
        {
            StatusCode = HttpStatusCode.OK,
            Content = new StringContent("[{""id"":1, ""name"":""Test User""}]")
        });

    var client = new HttpClient(mockHandler.Object);
    
    // Act
    var result = await YourClass.GetListAsync<User>(client, "http://test.com");
    
    // Assert
    Assert.That(result, Has.Count.EqualTo(1));
    Assert.That(result[0].Name, Is.EqualTo("Test User"));
}
```

Таким образом, работа со списками объектов в JSON практически не отличается от работы с одиночными объектами, главное правильно указать тип коллекции в generic-параметре.