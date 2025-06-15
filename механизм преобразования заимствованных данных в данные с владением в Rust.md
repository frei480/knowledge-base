---
tags:
  - rust
---
[[Основные понятия в Rust]]
# 🔑 Основные Механизмы "превращения В владельца"
## 1. **`.to_owned()`** (из [[Trait в Rust|трейта]] `ToOwned`)
Самый распространённый метод для получения владеющей копии данных:
```rust
// Для строк
let borrowed_str: &str = "Hello";
let owned_string: String = borrowed_str.to_owned();

// Для срезов
let slice: &[i32] = &[1, 2, 3];
let owned_vec: Vec<i32> = slice.to_owned();
```

**Как это работает:**
- Создаёт полную копию данных в куче
- Реализован для типов, реализующих [[Trait в Rust|трейт]] `Clone`
- Часто используется для преобразования `&str` → `String`

---

## 2. **`.clone()`** (из Трейта `Clone`)
Создаёт глубокую копию данных:
```rust
let original = String::from("Hello");
let cloned = original.clone(); // Полное копирование
```

**Отличие от `to_owned()`:**

| Метод         | Применимость                | Семантика                |
| ------------- | --------------------------- | ------------------------ |
| `.clone()`    | Любой `T: Clone`            | Глубокая копия           |
| `.to_owned()` | В основном для `dyn Borrow` | Может быть оптимизирован |

---

## 3. **`into()`** (из Трейта `Into`)
Преобразует тип через перемещение:
```rust
let s: &str = "text";
let owned: String = s.into(); // &str → String

let v: &[i32] = &[1, 2];
let owned: Vec<i32> = v.into(); // &[T] → Vec<T>
```

**Особенности:**
- Не копирует данные, а перемещает (если возможно)
- Использует [[владение и заимствование в rust|систему владения]] Rust

---


1. **Семантика владения** в Rust требует явности:
   - `.to_owned()` — "я хочу новую копию"
   - `.clone()` — "копирую существующие данные"
   - `into()` — "преобразую с возможным перемещением"

2. **Нет магических преобразований**:
   ```rust
   // Так НЕ работает!
   let borrowed = &42;
   let owned = borrowed.to_owner(); // Не существует!
   
   // Правильно:
   let owned = *borrowed; // Для Copy-типов
   let owned = borrowed.clone(); // Для Clone-типов
   ```

3. **Для примитивов** используется копирование:
   ```rust
   let num_ref = &5;
   let num_owned = *num_ref; // i32 реализует Copy
   ```

---

# 🛠️ Когда Что Использовать?
## Сценарий 1: Строки
```rust
let input: &str = "Привет";

// Вариант 1 (рекомендуется)
let s1: String = input.to_owned();

// Вариант 2
let s2: String = input.into();

// Вариант 3
let s3: String = input.to_string();
```

## Сценарий 2: Пользовательские Типы
```rust
#[derive(Clone)]
struct User {
    id: u64,
    name: String,
}

let user_ref = &User {
    id: 1,
    name: "Alice".to_owned(),
};

// Создаём владеющую копию
let user_owned = user_ref.clone();
```

---

# 🔄 Специальные Случаи
## Для Типов, Реализующих `Copy`
```rust
let x = 10;
let y = &x;
let z = *y; // Простое разыменование (не нужен to_owned)
```

## Для `CoW` (Copy-on-Write)
```rust
use std::borrow::Cow;

let data = "Строка";
let mut cow = Cow::Borrowed(data);

// Преобразуем в владельца только при необходимости
cow.to_mut().push_str(" изменённая");
```

---

# 💡 Лучшие Практики
1. **Используйте `to_owned()` для &str → String**
2. **Для срезов предпочитайте `to_vec()` или `into()`**
3. **Избегайте лишних копий** — работайте с ссылками где возможно
4. **Для конвертаций между типами** используйте трейты `From`/`Into`
   ```rust
   impl From<&str> for MyType {
       fn from(s: &str) -> Self {
           MyType(s.to_owned())
       }
   }
   
   let my_val: MyType = "text".into();
   ```

---

# ⚠️ Распространённая Ошибка
```rust
// Неправильно!
fn process(data: &str) -> String {
    data.to_owner() // Функции не существует!
}

// Правильно:
fn process(data: &str) -> String {
    data.to_owned()
}
```

---