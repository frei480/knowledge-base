---
tags:
  - rust
---
Отлично! Макросы в Rust — это мощный инструмент **метапрограммирования**, позволяющий генерировать код во время компиляции. Они бывают двух основных типов: **декларативные** (`macro_rules!`) и **процедурные** (custom derive, атрибутные, функциональные). Давайте разберём их детально.

---

# 📌 1. [[Декларативные макросы в Rust|Декларативные макросы]] (`macro_rules!`)
Самый распространённый тип. Работают через **сопоставление с образцом** (pattern matching).

## Базовый Синтаксис:
```rust
macro_rules! my_macro {
    // Паттерн => Код-результат
    ($x:expr) => {
        println!("Значение: {}", $x);
    };
    ($($x:expr),*) => {
        $(
            println!("{}", $x);
        )*
    };
}
```

## Пример: Знаменитый `vec!`
```rust
macro_rules! vec {
    () => { Vec::new() };  // Пустой вектор
    ($($x:expr),+ $(,)?) => {  // Один или более элементов
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}

// Использование
let v = vec![1, 2, 3];
```

## Ключевые Элементы:
- **Захватчики (captures):**
  `$x:expr` — выражение, `$i:ident` — идентификатор, `$t:ty` — тип
- **Повторители (repetitions):**
  `*` (0+ раз), `+` (1+ раз), `?` (0 или 1 раз)
- **Пример с HTML-билдером:**
  ```rust
  macro_rules! html {
      (<$tag>$content:tt</$closetag>) => { ... };
  }
  ```

---

# ⚙️ 2. [[Процедурные Макросы в Rust|Процедурные Макросы]]
Более мощные, но сложные. Реализуются как **отдельные крейты** (`proc-macro = true` в `Cargo.toml`).

## Три Подвида:
1. **[[Процедурные Макросы в Rust#🛠️ 1. Derive-макросы|Custom Derive Macros]]**
   Автоматически реализуют трейты для структур:
   ```rust
   #[derive(Serialize, Debug)]
   struct User { name: String }
   ```

2. **[[Процедурные Макросы в Rust#🏷️ 2. Атрибутные Макросы|Attribute-Like Macros]]**
   Применяются к блокам кода:
   ```rust
   #[route(GET, "/")]
   fn index() { ... }
   ```

3. **[[Процедурные Макросы в Rust#📞 3. Функциональные Макросы|Function-Like Macros]]**
   Выглядят как вызовы функций:
   ```rust
   let sql = sql!(SELECT * FROM users);
   ```

---

# 🛠️ Как Работают Процедурные Макросы?
1. **Вход:** `TokenStream` (исходный код)
2. **Преобразование:** В AST с помощью `syn`
3. **Генерация кода:** С помощью `quote`
4. **Выход:** Новый `TokenStream`

## Пример: Custom Derive Для `HelloMacro`
```rust
// Крейт proc-macro
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    let ast = parse_macro_input!(input as DeriveInput);
    let name = &ast.ident;
    
    quote! {
        impl HelloMacro for #name {
            fn hello() {
                println!("Hello from {}!", stringify!(#name));
            }
        }
    }.into()
}
```

**Использование:**
```rust
#[derive(HelloMacro)]
struct Pancakes;

fn main() {
    Pancakes::hello(); // "Hello from Pancakes!"
}
```

---

# 🌟 Популярные Макросы В Экосистеме
1. **`println!` / `format!`**
   Форматирование строк (система макросов сложнее `format_args!`)

2. **`lazy_static!`**
   Отложенная инициализация статических переменных:
   ```rust
   lazy_static! {
       static ref CONFIG: HashMap<String, String> = load_config();
   }
   ```

3. **`tokio::main`**
   Превращает async fn в точку входа:
   ```rust
   #[tokio::main]
   async fn main() { ... }
   ```

4. **`serde_derive`**
   Генерирует код для сериализации:
   ```rust
   #[derive(Serialize, Deserialize)]
   struct Data { ... }
   ```

---

# ⚠️ Ограничения И Подводные Камни
1. **Сложность отладки**
   Ошибки показываются в сгенерированном коде
2. **Время компиляции**
   Тяжёлые макросы замедляют сборку
3. **Порог входа**
   Требует глубокого понимания AST и токенов
4. **Правило гигиены**
   Макросы не "видят" внешние переменные:
   ```rust
   macro_rules! broken {
       () => { x + 1 }; // Ошибка: `x` не существует!
   }
   ```

---

# 💡 Best Practices
1. **Избегайте макросов, где хватит функций**
2. **Используйте `macro_rules!` для простых задач**
3. **Для сложной логики — процедурные макросы**
4. **Тестируйте через `cargo expand`**:
   ```bash
   cargo install cargo-expand
   cargo expand
   ```

---

# 🔧 Инструменты
1. **`cargo expand`** — просмотр развёрнутого кода
2. **`syn` + `quote`** — парсинг и генерация кода
3. **`proc-macro2`** — обратная совместимость

---

# 🎓 Пример: Макрос Для Тестирования
```rust
macro_rules! auto_test {
    ($name:ident($($arg:ty),*) -> $ret:ty) => {
        paste::item! {
            #[test]
            fn [< test_ $name >]() {
                let result = $name(<$arg>::default(), <$arg>::default());
                assert_eq!(result, <$ret>::default());
            }
        }
    };
}

auto_test!(add(i32, i32) -> i32); // Генерирует test_add
```

---

# 💎 Зачем Нужны Макросы?
1. **Устранение шаблонного кода** (как derive)
2. **Создание DSL** (Domain-Specific Languages)
   ```rust
   html! { <div class="header">{"Hello"}</div> }
   ```
3. **Статические проверки** как `sqlx::query!`
4. **Аннотации времени компиляции** (как `#[tokio::main]`)

---

# 📊 Сравнение С Другими Языками
| Особенность          | Rust              | C/C++            | Lisp             |
|----------------------|-------------------|------------------|------------------|
| Тип макросов         | Гигиеничные       | Текстовые        | AST-макросы      |
| Безопасность         | Высокая           | Низкая           | Зависит          |
| Сложность            | Средняя/высокая   | Низкая           | Высокая          |
| Интеграция с языком  | Идеальная         | Ограниченная     | Полная           |

---

# 🏁 Итог
Макросы в Rust — это:
- **Мощный инструмент метапрограммирования**
- **Два типа:** декларативные (проще) и процедурные (мощнее)
- **Ключевые применения:**
  - Устранение шаблонного кода
  - Создание встроенных DSL
  - Генерация кода из аннотаций
- **Лучшие друзья:** `syn`, `quote`, `cargo-expand`
