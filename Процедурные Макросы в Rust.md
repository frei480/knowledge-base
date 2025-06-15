---
tags:
  - rust
---
[[Основные понятия в Rust]]
Процедурные макросы в Rust — это мощный инструмент метапрограммирования, который позволяет выполнять произвольный код на этапе компиляции. В отличие от декларативных макросов (`macro_rules!`), процедурные макросы дают полную свободу за счёт работы с абстрактным синтаксическим деревом (AST). Давайте разберём их подробно.

---

# 📦 Основы Процедурных Макросов
Процедурные макросы реализуются в **отдельных [[Crate в Rust|крейтах]]** с указанием `proc-macro = true` в `Cargo.toml`:

```toml
[lib]
proc-macro = true

[dependencies]
syn = "2.0"
quote = "1.0"
proc-macro2 = "1.0"
```

## Три Типа Процедурных Макросов:
1. **Derive-макросы** (наиболее распространённые)
2. **Атрибутные макросы**
3. **Функциональные макросы**

---

# 🛠️ 1. Derive-макросы
Автоматически реализуют трейты для структур и перечислений.

## Пример: Генерация Builder-паттерна
**Цель:** Автоматическое создание строителя для структуры.

```rust
#[derive(Builder)]
struct User {
    id: i32,
    name: String,
    email: Option<String>,
}

// Автоматически генерирует:
impl User {
    pub fn builder() -> UserBuilder {
        UserBuilder::default()
    }
}
```

### Реализация (`src/lib.rs`):
```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

#[proc_macro_derive(Builder)]
pub fn derive_builder(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    let name = &input.ident;
    let builder_name = syn::Ident::new(&format!("{}Builder", name), name.span());

    // Генерация полей для билдера
    let fields = if let syn::Data::Struct(syn::DataStruct {
        fields: syn::Fields::Named(ref fields),
        ..
    }) = input.data
    {
        &fields.named
    } else {
        panic!("Only structs with named fields are supported");
    };

    let builder_fields = fields.iter().map(|f| {
        let name = &f.ident;
        let ty = &f.ty;
        quote! { #name: std::option::Option<#ty> }
    });

    // Генерация методов set_*
    let set_methods = fields.iter().map(|f| {
        let name = &f.ident;
        let ty = &f.ty;
        quote! {
            pub fn #name(&mut self, value: #ty) -> &mut Self {
                self.#name = Some(value);
                self
            }
        }
    });

    // Генерация метода build
    let build_checks = fields.iter().map(|f| {
        let name = &f.ident;
        quote! {
            #name: self.#name.clone().ok_or(format!("Field {} is missing", stringify!(#name)))?
        }
    });

    let expanded = quote! {
        #[derive(Default)]
        struct #builder_name {
            #(#builder_fields,)*
        }

        impl #builder_name {
            #(#set_methods)*

            pub fn build(&mut self) -> Result<#name, String> {
                Ok(#name {
                    #(#build_checks,)*
                })
            }
        }

        impl #name {
            pub fn builder() -> #builder_name {
                #builder_name::default()
            }
        }
    };

    expanded.into()
}
```

---

# 🏷️ 2. Атрибутные Макросы
Изменяют аннотируемый элемент, добавляя или изменяя функциональность.

## Пример: Логирование Вызовов Функции
```rust
#[log_calls]
fn expensive_operation(n: i32) -> i32 {
    // Длительная операция
    n * 2
}
```

### Реализация:
```rust
#[proc_macro_attribute]
pub fn log_calls(_attr: TokenStream, item: TokenStream) -> TokenStream {
    let input = parse_macro_input!(item as syn::ItemFn);
    let fn_name = &input.sig.ident;
    let block = &input.block;

    let expanded = quote! {
        fn #fn_name() -> i32 {
            println!("[LOG] Calling {}", stringify!(#fn_name));
            let start = std::time::Instant::now();
            let result = #block;
            println!("[LOG] {} completed in {:?}", stringify!(#fn_name), start.elapsed());
            result
        }
    };

    expanded.into()
}
```

---

# 📞 3. Функциональные Макросы
Выглядят как вызовы функций и работают с произвольными токенами.

## Пример: SQL-запросы С Проверкой На Этапе Компиляции
```rust
let query = sql!(SELECT * FROM users WHERE age > 10);
```

### Реализация:
```rust
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as sql_macro::Input);
    
    // Проверка синтаксиса SQL и генерация кода
    let validated_query = validate_sql(&input);
    
    quote! {
        {
            use sqlx::query;
            query(#validated_query)
        }
    }.into()
}
```

---

# 🔍 Ключевые Компоненты Для Работы

## 1. `syn` — Парсинг Входных Данных
Позволяет разбирать `TokenStream` в структурированные AST-узлы:
```rust
use syn::{parse_macro_input, DeriveInput, ItemFn};

// Для derive-макросов
let input = parse_macro_input!(input as DeriveInput);

// Для функций
let input = parse_macro_input!(input as ItemFn);
```

## 2. `quote` — Генерация Кода
Преобразует Rust-код обратно в `TokenStream`:
```rust
let name = syn::Ident::new("MyStruct", proc_macro2::Span::call_site());
let output = quote! {
    impl #name {
        fn new() -> Self {
            Self
        }
    }
};
```

## 3. `proc_macro2` — Совместимость
Работа с токенами вне контекста макроса:
```rust
use proc_macro2::TokenStream;
let tokens: TokenStream = /* ... */;
```

---

# 🧪 Тестирование Процедурных Макросов
Используйте `proc_macro2` и `syn` для модульного тестирования:

```rust
#[test]
fn test_builder_macro() {
    let input = r#"
        #[derive(Builder)]
        struct User {
            id: i32,
            name: String,
        }
    "#;
    
    let parsed = syn::parse_str(input).unwrap();
    let output = derive_builder(parsed.into());
    
    // Проверка сгенерированного кода
    assert!(output.to_string().contains("struct UserBuilder"));
}
```

---

# ⚠️ Ограничения И Сложности
1. **Отладка**: Ошибки показываются в сгенерированном коде
2. **Время компиляции**: Тяжёлые макросы замедляют сборку
3. **Стабильность**: API `proc_macro` пока не стабилизировано полностью
4. **Сложность AST**: Требует глубокого понимания структуры кода Rust

---

# 🚀 Лучшие Практики
1. **Разделяйте логику**: Выносите сложную логику в отдельные функции
2. **Генерация качественных ошибок**:
   ```rust
   syn::Error::new_spanned(field, "Unsupported field type")
       .to_compile_error()
       .into()
   ```
3. **Используйте помощники**:
   - `darling` для упрощения работы с атрибутами
   - `heck` для преобразования именования
4. **Кэшируйте**: Генерируйте код только при необходимости

---

# 💡 Продвинутые Техники

## 1. Сохранение Позиций В Коде
Используйте `Span` для корректных сообщений об ошибках:
```rust
let span = input.ident.span();
syn::Error::new(span, "Custom error message").to_compile_error().into()
```

## 2. Обработка Generic-типов
```rust
let generics = &input.generics;
let (impl_generics, ty_generics, where_clause) = generics.split_for_impl();

quote! {
    impl #impl_generics MyTrait for #name #ty_generics #where_clause {
        // ...
    }
}
```

## 3. Работа С Атрибутами
```rust
#[derive(Builder, Debug, PartialEq)]
#[builder(verbose)]
struct User { /* ... */ }
```

Извлечение атрибутов:
```rust
for attr in &input.attrs {
    if attr.path().is_ident("builder") {
        // Обработка атрибута
    }
}
```

---

# 🌟 Реальные Примеры Использования
1. **`serde`**: `#[derive(Serialize, Deserialize)]`
2. **`tokio`**: `#[tokio::main]`
3. **`async-trait`**: Преобразование трейтов в асинхронные
4. **`sqlx`**: Проверка SQL-запросов на этапе компиляции
5. **`mockall`**: Автоматическое создание мок-объектов

---

# 📊 Сравнение С Декларативными Макросами
| Критерий               | Процедурные макросы          | `macro_rules!`              |
|------------------------|-------------------------------|-----------------------------|
| Мощность               | Полная по Тьюрингу            | Ограниченная                |
| Сложность              | Высокая                       | Умеренная                  |
| Работа с AST           | Полный доступ                 | Только паттерн-матчинг     |
| Время компиляции       | Значительное                  | Минимальное                |
| Отладка                | Сложная                       | Относительно простая       |
| Переиспользование кода | Полное (как обычный код Rust) | Ограниченное               |

---

# 🏁 Итог
Процедурные макросы в Rust — это мощный инструмент, который позволяет:
1. Создавать сложные DSL (предметно-ориентированные языки)
2. Автоматически генерировать шаблонный код
3. Выполнять сложные проверки на этапе компиляции
4. Расширять возможности языка

Для эффективной работы:
1. Используйте `syn` и `quote`
2. Тестируйте с помощью `proc-macro2`
3. Следуйте лучшим практикам генерации ошибок
4. Изучайте AST через `cargo expand`
