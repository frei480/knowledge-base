---
tags:
  - javascript
---
Вот как правильно писать асинхронный код с обработкой HTTP-запросов на современном JavaScript:

**1. Используйте async/await вместо колбэков**
```javascript
// Плохой подход (колбэки)
fetch('https://api.example.com/data', (error, response) => {
  if (error) console.error(error);
  else processData(response);
});

// Хороший подход (async/await)
async function fetchData() {
  try {
    const response = await fetch('https://api.example.com/data');
    const data = await response.json();
    processData(data);
  } catch (error) {
    console.error('Ошибка:', error);
  }
}
```

**2. Обрабатывайте ошибки с try/catch**
```javascript
async function getData() {
  try {
    const response = await fetch('https://api.example.com/data');
    if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
    return await response.json();
  } catch (error) {
    console.error('Ошибка получения данных:', error);
    throw error; // Прокидываем ошибку дальше при необходимости
  }
}
```

**3. Параллельные запросы с Promise.all**
```javascript
async function fetchMultiple() {
  try {
    const [users, posts] = await Promise.all([
      fetch('/api/users').then(r => r.json()),
      fetch('/api/posts').then(r => r.json())
    ]);
    return {users, posts};
  } catch (error) {
    console.error('Один из запросов не удался:', error);
  }
}
```

**4. Последовательные запросы**
```javascript
async function sequentialRequests() {
  const user = await fetchUser();
  const posts = await fetchUserPosts(user.id);
  return {user, posts};
}
```

**5. Используйте современные библиотеки**
Пример с axios:
```javascript
async function fetchWithAxios() {
  try {
    const response = await axios.get('https://api.example.com/data');
    return response.data;
  } catch (error) {
    if (error.response) {
      // Ошибка сервера (5xx, 4xx)
      console.error('Server error:', error.response.status);
    } else {
      console.error('Network error:', error.message);
    }
  }
}
```

**6. Работа с несколькими источниками данных**
```javascript
async function getCombinedData() {
  try {
    const [data1, data2] = await Promise.all([
      fetchDataFromSource1(),
      fetchDataFromSource2()
    ]);
    
    return {
      combined: data1.results.concat(data2.results),
      meta: {count: data1.count + data2.count}
    };
  } catch (error) {
    console.error('Ошибка комбинирования данных:', error);
    return {combined: [], meta: {count: 0}};
  }
}
```

**Важные рекомендации:**
1. Всегда обрабатывайте ошибки с помощью try/catch
2. Используйте Promise.all для параллельных запросов
3. Избегайте излишней вложенности асинхронных операций
4. Для Node.js используйте последние версии (18+) с нативным fetch
5. Используйте деструктуризацию для упрощения кода
6. Добавляйте проверки ответов сервера:
```javascript
if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
```

**Пример полного цикла запроса:**
```javascript
async function fetchAndProcess() {
  try {
    // Запрос данных
    const response = await fetch('https://api.example.com/data');
    
    // Проверка статуса
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    // Парсинг JSON
    const data = await response.json();
    
    // Обработка данных
    const processed = processData(data);
    
    // Возврат результата
    return processed;
  } catch (error) {
    // Логирование ошибки
    console.error('Fetch failed:', error);
    
    // Возврат значения по умолчанию
    return {default: true};
  }
}
```

Эти подходы помогут писать чистый, поддерживаемый асинхронный код с правильной обработкой ошибок и эффективным управлением параллельными операциями.