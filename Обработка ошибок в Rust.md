---
tags:
  - rust
---
[[Основные понятия в Rust]]
Обработка ошибок в Rust — это одна из сильнейших сторон языка, основанная на **явности и безопасности типов**. В отличие от исключений в других языках, Rust использует комбинацию типов `Result` и `Option`, что гарантирует обработку ошибок на этапе компиляции. Разберём ключевые аспекты.

---

# 📌 Основные Типы Для Обработки Ошибок
## 1. `Option<T>` — Для Отсутствующих Значений
```rust
enum Option<T> {
    Some(T),
    None,
}
```
- Используется, когда значение может отсутствовать (без ошибки как таковой)
- Пример: поиск элемента в коллекции
```rust
let numbers = vec![1, 2, 3];
let maybe_num = numbers.get(5); // Возвращает Option<&i32>
```

## 2. `Result<T, E>` — Для Обработки Ошибок
```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
- `T` — тип успешного результата
- `E` — тип ошибки (должен реализовывать трейт `std::error::Error`)
```rust
use std::fs::File;

let file_result = File::open("config.toml");
// file_result: Result<File, std::io::Error>
```

---

# 🛠️ Основные Методы Работы
## 1. Явная Обработка Через `match`
```rust
let file = match File::open("config.toml") {
    Ok(f) => f,
    Err(e) => {
        eprintln!("Ошибка открытия файла: {}", e);
        return; // или обработка ошибки
    }
};
```

## 2. "Сахарки" Для Быстрой Обработки
- `unwrap()`: Паника при ошибке (только для прототипов)
  ```rust
  let file = File::open("config.toml").unwrap();
  ```
- `expect()`: Паника с кастомным сообщением
  ```rust
  let file = File::open("config.toml").expect("Файл конфига обязателен");
  ```
- `unwrap_or_default()`: Возвращает значение по умолчанию
  ```rust
  let num: i32 = "123".parse().unwrap_or_default(); // 123
  let num: i32 = "abc".parse().unwrap_or_default(); // 0
  ```

## 3. Проброс Ошибок С `?`
Оператор `?` автоматически возвращает ошибку из функции:
```rust
use std::io::{self, Read};

fn read_config() -> Result<String, io::Error> {
    let mut file = File::open("config.toml")?; // Если ошибка - сразу возвращаем Err
    let mut contents = String::new();
    file.read_to_string(&mut contents)?; // Аналогично
    Ok(contents)
}
```

---

# 📦 Расширенная Обработка Ошибок
## 1. Преобразование Ошибок
```rust
fn parse_config() -> Result<Config, String> {
    let contents = read_config().map_err(|e| format!("Ошибка чтения: {}", e))?;
    contents.parse().map_err(|e| format!("Ошибка парсинга: {}", e))
}
```

## 2. Цепочки Вызовов С `and_then`
```rust
"123"
    .parse::<i32>()
    .and_then(|n| if n > 0 { Ok(n) } else { Err("Число должно быть положительным".into()) })
    .and_then(|n| calculate(n))
```

---

# 🧩 Создание Собственных Типов Ошибок
## Вариант 1: Через `thiserror` (рекомендуется)
```rust
use thiserror::Error;

#[derive(Error, Debug)]
enum ConfigError {
    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),
    #[error("Parse error: {0}")]
    Parse(#[from] serde_json::Error),
    #[error("Validation error: {field} {message}")]
    Validation { field: String, message: String },
}
```

## Вариант 2: Вручную
```rust
use std::fmt;

#[derive(Debug)]
struct MyError {
    details: String,
}

impl std::error::Error for MyError {}

impl fmt::Display for MyError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{}", self.details)
    }
}
```

---

# 🔄 Комбинирование `Option` И `Result`
## Преобразование `Option` В `Result`
```rust
let maybe_num: Option<i32> = Some(42);
let result: Result<i32, &str> = maybe_num.ok_or("Значение отсутствует");
```

## Обработка Через `ok_or_else`
```rust
let num = numbers
    .get(5)
    .ok_or_else(|| MyError::new("Индекс за границами"))?;
```

---

# 🚀 Best Practices
1. **Всегда используйте `Result` для функций, которые могут завершиться ошибкой**
2. **Избегайте `unwrap()`/`expect()` в продакшн-коде**
3. **Для приложений используйте `anyhow`**:
   ```rust
   use anyhow::{Context, Result};
   
   fn main() -> Result<()> {
       let data = std::fs::read_to_string("file.txt")
           .context("Не удалось прочитать файл")?;
       // ...
       Ok(())
   }
   ```
4. **Для библиотек используйте `thiserror`**
5. **Логируйте цепочки ошибок**:
   ```rust
   eprintln!("Ошибка: {:#}", err); // Красивый вывод вложенных ошибок
   ```

---

# 🌟 Пример: Полный Пайплайн Обработки
```rust
use anyhow::{Context, Result};
use serde::Deserialize;

#[derive(Deserialize)]
struct Config { api_key: String }

fn load_config() -> Result<Config> {
    let path = "config.json";
    let data = std::fs::read_to_string(path)
        .with_context(|| format!("Не удалось прочитать {}", path))?;
    
    serde_json::from_str(&data)
        .with_context(|| format!("Ошибка парсинга {}", path))
}

fn main() -> Result<()> {
    let config = load_config()?;
    println!("API Key: {}", config.api_key);
    Ok(())
}
```

---

# 💡 Почему Этот Подход Лучше Исключений?
1. **Явность** — сигнатуры функций показывают возможные ошибки
2. **Безопасность** — компилятор заставляет обрабатывать ошибки
3. **Производительность** — отсутствие накладных расходов на стек исключений
4. **Чёткий контроль потока** — ошибки обрабатываются как обычные значения

---

# 📊 Сравнение Подходов
| Метод               | Плюсы                          | Минусы                 |
|---------------------|--------------------------------|------------------------|
| `match`             | Полный контроль                | Многословный           |
| `?`                 | Лаконичный, безопасный         | Требует возврата Result|
| `unwrap()`          | Быстро для прототипов          | Паники в рантайме      |
| `thiserror`/`anyhow`| Профессиональная обработка     | Дополнительные зависимости |

Обработка ошибок в Rust — это философия **явности и безопасности**. Хотя сначала подход может показаться непривычным, он приводит к более надёжному и предсказуемому коду. Хотите углубимся в конкретные кейсы или нюансы?