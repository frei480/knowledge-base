---
tags:
  - rust
---
[[Основные понятия в Rust]]
Давайте глубоко погрузимся в **декларативные макросы** Rust (также известные как `macro_rules!`). Это мощный инструмент для метапрограммирования, позволяющий создавать гибкие шаблоны кода. Разберём всё по полочкам.

---

# 📐 Базовый Синтаксис
```rust
macro_rules! имя_макроса {
    // Паттерны для сопоставления
    (шаблон) => { генерируемый_код };
    (другой_шаблон) => { другой_код };
    // ...
}
```

---

# 🧩 Основные Компоненты

## 1. **Захватчики (Captures)**
Специальные маркеры для захвата частей кода:

| Маркер      | Тип данных             | Пример                     |
|-------------|------------------------|----------------------------|
| `$x:ident`  | Идентификатор          | `x`, `count`, `MyStruct`  |
| `$e:expr`   | Выражение              | `2+2`, `func()`           |
| `$t:ty`     | Тип                    | `i32`, `Vec<String>`      |
| `$i:item`   | Элемент кода           | `struct Foo;`, `fn bar()` |
| `$b:block`  | Блок кода              | `{ let x = 1; x }`        |
| `$p:pat`    | Паттерн                | `Some(x)`, `Point{x,y}`   |
| `$l:literal`| Литерал                | `42`, `"hello"`           |
| `$tt:tt`    | Токенное дерево (любое)| `+`, `[ ]`, `::`, `foo`   |

Пример:
```rust
macro_rules! log_var {
    ($var:ident) => {
        println!("{} = {:?}", stringify!($var), $var);
    };
}
```

## 2. **Повторители (Repetitions)**
Обработка повторяющихся элементов:

```rust
// Синтаксис:
$($element:capture),*  // 0 или более раз
$($element:capture),+  // 1 или более раз
$($element:capture)?   // 0 или 1 раз
```

Пример создания вектора:
```rust
macro_rules! my_vec {
    ($($x:expr),*) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}

let v = my_vec![1, 2, 3];
```

## 3. **Сепараторы (Separators)**
Могут быть любыми токенами:
```rust
($($x:expr),*)     // Запятая как разделитель
($($key:expr => $value:expr),+)  // Стрелка для map
```

---

# 🔍 Паттерн-матчинг
Макросы выбирают ветку по **первому совпадающему** паттерну. Порядок важен!

Пример с обработкой разных случаев:
```rust
macro_rules! calculate {
    // Одиночное число
    ($x:literal) => { $x };
    
    // Сложение
    ($a:literal + $b:literal) => { $a + $b };
    
    // Умножение с приоритетом
    (($($inner:tt)*) * $b:literal) => {
        calculate!($($inner)*) * $b
    };
}

let a = calculate!(5);         // 5
let b = calculate!(2 + 3);     // 5
let c = calculate!((2 + 3) * 4); // 20
```

---

# 🛠️ Практические Примеры

## 1. DSL Для HTML
```rust
macro_rules! html {
    // Базовый элемент с атрибутами и содержимым
    (<$tag:ident $($attr:ident=$value:expr)*>$($content:tt)*</$closetag:ident>) => {
        // Проверка совпадения тегов
        assert_eq!(stringify!($tag), stringify!($closetag));
        format!(
            "<{} {}>{}</{}>",
            stringify!($tag),
            $(
                format!("{}=\"{}\"", stringify!($attr), $value),
            )*
            html!($($content)*),
            stringify!($tag)
        )
    };
    
    // Текстовое содержимое
    ($text:literal) => { $text };
}

let page = html!(
    <div class="container">
        <h1>"Hello!"</h1>
    </div>
);
```

## 2. Автоматическая Генерация Тестов
```rust
macro_rules! generate_tests {
    ($($name:ident: $input:expr => $expected:expr,)*) => {
        $(
            #[test]
            fn $name() {
                assert_eq!(process($input), $expected);
            }
        )*
    }
}

generate_tests! {
    test_zero: 0 => 0,
    test_positive: 42 => 42,
    test_negative: -5 => 5,
}
```

## 3. Безопасные SQL-запросы
```rust
macro_rules! sql {
    (SELECT $($col:ident),+ FROM $table:ident WHERE $field:ident = $val:expr) => {
        {
            let query = format!(
                "SELECT {} FROM {} WHERE {} = ?",
                [$($col),*].join(", "),
                stringify!($table),
                stringify!($field)
            );
            execute_query(query, $val)
        }
    }
}

let result = sql!(SELECT id, name FROM users WHERE email = "test@example.com");
```

---

# ⚠️ Особенности И Подводные Камни

## 1. Гигиена Макросов
Rust автоматически избегает конфликтов имён:
```rust
macro_rules! create_var {
    () => {
        let x = 42;
    };
}

fn main() {
    let x = 10;
    create_var!(); // Не конфликтует с внешней x
    println!("{}", x); // 10, а не 42!
}
```

## 2. Рекурсивные Макросы
Макросы могут вызывать сами себя:
```rust
macro_rules! factorial {
    (0) => { 1 };
    ($n:expr) => {
        $n * factorial!($n - 1)
    };
}

let fact_5 = factorial!(5); // 120
```

## 3. Отладка
Используйте `cargo expand` для просмотра сгенерированного кода:
```bash
$ cargo install cargo-expand
$ cargo expand
```

---

# 🎓 Продвинутые Техники

## 1. Перегрузка По Разным Паттернам
```rust
macro_rules! multi_dispatch {
    ($($arg:tt)*) => {
        match $($arg)* {
            // Обработка разных типов аргументов
        }
    };
}
```

## 2. Внутренние Правила (Internal Rules)
Используйте `@` для внутренних шаблонов:
```rust
macro_rules! json {
    // Публичный интерфейс
    ($($json:tt)+) => {
        json_parse!(@value $($json)+)
    };
    
    // Внутренние правила
    (@value null) => { JsonValue::Null };
    (@value true) => { JsonValue::Boolean(true) };
    // ...
}
```

## 3. Работа С Разными Разделителями
```rust
macro_rules! list {
    ($($element:expr),*) => { ... };
    ($($element:expr),+ ,) => { ... }; // Обработка завершающей запятой
}
```

---

# 📊 Сравнение С Процедурными Макросами

| Характеристика       | `macro_rules!`               | Процедурные макросы       |
|----------------------|------------------------------|---------------------------|
| Сложность            | Низкая-средняя               | Высокая                   |
| Мощность             | Ограниченная                 | Полная по Тьюрингу        |
| Поддержка IDE        | Хорошая                      | Ограниченная              |
| Время компиляции     | Быстрое                      | Медленное                 |
| Обработка ошибок     | Базовые                      | Расширенные               |
| Гибкость             | Шаблонный подход             | Любая логика              |

---

# 💡 Best Practices
1. **Начинайте с простых паттернов**
2. **Используйте `tt` для сложных случаев**
3. **Добавляйте комментарии к правилам**
4. **Тестируйте с `cargo expand`**
5. **Избегайте излишней сложности**

```rust
// Плохо: слишком сложно
($a:expr, $b:expr, $c:expr, $d:expr) => { ... }

// Лучше: используйте повторители
($($arg:expr),*) => { ... }
```

---

# 🏁 Итог
Декларативные макросы (`macro_rules!`) — это мощный инструмент для:
- Устранения шаблонного кода
- Создания DSL (предметно-ориентированных языков)
- Генерации кода на этапе компиляции
- Повышения выразительности кода

Хотя они имеют ограничения по сравнению с процедурными макросами, их проще использовать, и они отлично подходят для большинства задач метапрограммирования в Rust.
