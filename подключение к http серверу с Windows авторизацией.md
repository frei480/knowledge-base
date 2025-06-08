---
tags:
  - csharp
---

Чтобы подключиться к серверу по HTTP с использованием авторизации Windows в C#, вы можете использовать следующие подходы:

### 1. Использование HttpClient (рекомендуемый способ)
```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        try
        // Указываем использовать учетные данные Windows
        using (var handler = new HttpClientHandler { UseDefaultCredentials = true })
        using (var client = new HttpClient(handler))
        {
            // Отправляем GET-запрос
            HttpResponseMessage response = await client.GetAsync("http://your-server.com/resource");
            
            // Проверяем успешность запроса
            response.EnsureSuccessStatusCode();
            
            // Читаем ответ
            string content = await response.Content.ReadAsStringAsync();
            Console.WriteLine(content);
        }
        catch (HttpRequestException ex)
        {
            Console.WriteLine($"Ошибка: {ex.Message}");
        }
    }
}
```

### 2. Использование WebClient (устаревший, но простой способ)
```csharp
using System;
using System.Net;

class Program
{
    static void Main()
    {
        try
        {
            using (var client = new WebClient())
            {
                client.UseDefaultCredentials = true; // Используем текущие учетные данные
                string result = client.DownloadString("http://your-server.com/resource");
                Console.WriteLine(result);
            }
        }
        catch (WebException ex)
        {
            Console.WriteLine($"Ошибка: {ex.Message}");
        }
    }
}
```

### 3. Использование HttpWebRequest (базовый уровень)
```csharp
using System;
using System.Net;

class Program
{
    static void Main()
    {
        try
        {
            HttpWebRequest request = (HttpWebRequest)WebRequest.Create("http://your-server.com/resource");
            request.Credentials = CredentialCache.DefaultCredentials; // Авторизация Windows
            
            using (HttpWebResponse response = (HttpWebResponse)request.GetResponse())
            using (System.IO.Stream stream = response.GetResponseStream())
            using (System.IO.StreamReader reader = new System.IO.StreamReader(stream))
            {
                Console.WriteLine(reader.ReadToEnd());
            }
        }
        catch (WebException ex)
        {
            Console.WriteLine($"Ошибка: {ex.Message}");
        }
    }
}
```

### Важные моменты:
1. **UseDefaultCredentials** - использует учетные данные текущего пользователя Windows
2. Для работы необходимо:
   - Сервер должен поддерживать Integrated Windows Authentication (NTLM/Kerberos)
   - Клиент и сервер должны быть в одном домене или доверенных доменах
   - На сервере должна быть настроена соответствующая авторизация

Если нужно указать явные учетные данные (например, другого пользователя домена):
```csharp
var credentials = new NetworkCredential("username", "password", "domain");
var handler = new HttpClientHandler { Credentials = credentials };
```

Для HTTPS-соединений с самоподписанными сертификатами добавьте:
```csharp
handler.ServerCertificateCustomValidationCallback = 
    (sender, cert, chain, sslPolicyErrors) => { return true; };
```

Все примеры включают базовую обработку ошибок. Для продакшен-кода рекомендуется более детальная обработка исключений.