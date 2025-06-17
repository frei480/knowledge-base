---
tags:
  - rust
---
[[Основные понятия в Rust]]
Алгебраические типы данных (АТД) — одна из **сильнейших сторон Rust**, позволяющая выражать сложные структуры данных безопасно и элегантно. Давайте разберём их детально.

# 📐 Что Такое Алгебраические Типы Данных?
АТД — концепция из теории типов, объединяющая:
1. **Типы-произведения (product types)** — например, структуры (`struct`), кортежи (`tuple`).
2. **Типы-суммы (sum types)** — например, перечисления (`enum`).

Rust реализует АТД через:
- `struct` (композиция полей)
- `enum` (вариантность)
- `tuple` (анонимная композиция)

---

# 🧩 Типы-произведения (Product Types)
## 1. Структуры (`struct`)
```rust
// Именованная структура
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

// Использование
let user1 = User {
    email: String::from("user@example.com"),
    username: String::from("username123"),
    active: true,
    sign_in_count: 1,
};
```

## 2. Кортежные Структуры (Tuple Structs)
```rust
struct Color(u8, u8, u8);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

## 3. Единичная Структура (Unit Struct)
```rust
struct AlwaysEqual;
let subject = AlwaysEqual;
```

---

# 🎲 Типы-суммы (Sum Types)
## Перечисления (`enum`)
Главная сила Rust — `enum`, которые могут содержать данные:
```rust
enum WebEvent {
    PageLoad,                 // Вариант без данных
    KeyPress(char),           // Одно значение
    Paste(String),            // String
    Click { x: i64, y: i64 }, // Анонимная структура
}
```

## Пример С Вложенными Типами
```rust
enum Shape {
    Circle { radius: f64 },
    Rectangle { width: f64, height: f64 },
    Triangle(Point, Point, Point), // Кортежный вариант
}
```

---

# 💡 Почему АТД В Rust Мощные?
## 1. Паттерн-матчинг (обязательная Обработка Всех вариантов)
```rust
fn handle_event(event: &WebEvent) -> String {
    match event {
        WebEvent::PageLoad => "Page loaded".to_string(),
        WebEvent::KeyPress(c) => format!("Pressed key: {}", c),
        WebEvent::Paste(s) => format!("Pasted text: {}", s),
        WebEvent::Click { x, y } => format!("Clicked at ({}, {})", x, y),
    }
}
```

## 2. Комбинация Struct + Enum
```rust
struct UiElement {
    id: String,
    position: (i32, i32),
    element_type: ElementType, // enum внутри struct!
}

enum ElementType {
    Button { label: String },
    TextField { placeholder: String },
    Checkbox(bool),
}
```

## 3. Рекурсивные Типы (через `Box`)
```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

// Пример списка: 1 -> 2 -> 3 -> Nil
let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
```

---

# 🌟 Знаковые АТД Из Стандартной Библиотеки
## 1. `Option<T>` (избегаем `null`)
```rust
enum Option<T> {
    Some(T),
    None,
}

fn safe_division(numerator: f64, denominator: f64) -> Option<f64> {
    if denominator == 0.0 {
        None
    } else {
        Some(numerator / denominator)
    }
}
```

## 2. `Result<T, E>` (обработка ошибок)
```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}

fn read_file(path: &str) -> Result<String, io::Error> {
    fs::read_to_string(path)
}
```

## 3. `IpAddr` (реальный Пример Из std)
```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));
```

---

# 🔧 Методы Для АТД
Можно добавлять методы через `impl`:
```rust
impl WebEvent {
    fn log(&self) {
        match self {
            Self::PageLoad => println!("Page loaded"),
            // ... другие варианты ...
        }
    }
}

let event = WebEvent::KeyPress('x');
event.log();
```

---

# ⚠️ Особенности И Ограничения
1. **Размер enum** определяется по **наибольшему варианту**:
   ```rust
   // Занимает 24 байта (String), даже для Empty!
   enum Data {
       Empty,
       Text(String), // String = 24 байта
   }
   ```

2. **Для рекурсивных типов всегда нужен `Box`** (или другой умный указатель), чтобы размер был известен компилятору.

3. **Паттерн-матчинг должен быть исчерпывающим**:
   ```rust
   match value {
       Some(x) => {}, // Ошибка! Не обработан None
   }
   ```

---

# 💡 Советы По Использованию
1. **Предпочитайте `match` `if let`** для безопасности
2. **Используйте вложенные enum** для сложных состояний:
   ```rust
   enum ConnectionState {
       Disconnected,
       Connecting { attempts: u32 },
       Connected { session_id: Uuid },
   }
   ```
3. **Применяйте для конечных автоматов** (state machines)

---

# 🆚 Сравнение С Другими Языками
| Особенность         | Rust            | Java/C#       | C/C++         |
|---------------------|-----------------|---------------|---------------|
| Sum types           | Да (`enum`)     | Нет           | Нет           |
| Pattern matching    | Полноценный     | Ограниченный  | Отсутствует   |
| Null-безопасность   | `Option<T>`     | `Nullable`    | Указатели     |
| Размещение данных   | В стеке         | Куча          | Куча/стек     |

---

# 💎 Пример: Парсер JSON
```rust
enum JsonValue {
    Null,
    Boolean(bool),
    Number(f64),
    String(String),
    Array(Vec<JsonValue>),
    Object(HashMap<String, JsonValue>),
}

fn parse_json(input: &str) -> Result<JsonValue, ParseError> {
    // ... реализация с использованием АТД ...
}
```

# 🏁 Итог
Алгебраические типы данных в Rust — это:
- **Мощный инструмент моделирования** через `enum` + `struct`
- **Безопасность** через паттерн-матчинг
- **Эффективность** (размер известен на этапе компиляции)
- **Ясность кода** (явное выражение состояний)

АТД лежат в основе многих возможностей Rust и помогают избежать целых классов ошибок.
Разберём [[АТД в Rust предотвращение ошибок на этапе компиляции|конкретный пример применения]].