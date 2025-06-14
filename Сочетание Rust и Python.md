---
tags:
  - python
  - rust
---
пересказ видео https://www.youtube.com/watch?v=lyG6AKzu4ew
Автор: Arjan Egges
Сочетание Rust и Python: Лучшее из Обоих Миров?

00:00 Введение

• Python известен своей дружественностью, но не скоростью и безопасностью типов.
• Rust предлагает высокую скорость и безопасность типов.
• Видео покажет, как объединить Python и Rust для улучшения кода.

00:27 Настройка проекта

• Пример библиотеки на Rust с простыми функциями.
```rust
use pyo3::prelude::*;
///выворд суммы двух чисел в виде строки
#[pyfunction]
fn sum_as_string(a: usize, b: usize) -> PyResult<String>{
	Ok((a+b).to_string())
}
#[pymodule]
fn pyo3_rust(_py: Python, m: &PyModule) -> PyResult<()> {
	m.add_function(wrap_pyfunction!(sum_as_string, m)?)?;
	Ok(())
}
```
• Необходимость исходной папки, файла `cargo.toml` и файла проекта `pyproject.toml`.
```toml
#cargo.toml
[package]
name = "pyo3_rust"
version = "0.1.0"
edition = "2021"

[lib]
name = "pyo3_rust"
crate-type = ["cdylib"]

[dependencies]
pyo3 = "0.19.0"
```

```toml
#pyproject.toml
[build-system]
requires = ["maturin>=1.4,<2.0"]
build-backend = "maturin"

[project]
name = "pyo3_rust"
requires-python = ">=3.8"
classifiers = [
    "Programming Language :: Rust",
    "Programming Language :: Python :: Implementation :: CPython",
    "Programming Language :: Python :: Implementation :: PyPy",
]
dynamic = ["version"]

[dependencies]
maturin = "1.4.0"
rustimport = "1.4.0"

[tool.maturin]
features = ["pyo3/extension-module"]
```

• Использование pyo3 для создания модулей Python на основе Rust.

01:10 Pyo3 и его особенности

• Pyo3 позволяет писать модули Python на основе Rust.
• Rust-код может быть оптимизирован и безопасен для памяти.
• Pyo3 определяет теги py module и py function для связи с Python.

02:22 Обработка ошибок и примеры

• Rust обрабатывает ошибки иначе, чем Python.
• Пример функции sum_as_string, которая складывает два числа и преобразует их в строку.
• Определение модуля с функцией sum_as_string для импорта в Python.

03:22 Использование Maturin

• Maturin помогает создавать и развертывать модули Python на основе Rust.
```shell
pip install maturin
```
создание проекта:
```shell
maturin init
```
компиляция кода:
```shell
maturin develop
```
• Сравнение с Poetry и использование файла project.toml.
• Maturin поддерживает другие функции, такие как CFI и CPython.

05:43 Примеры использования Pyo3

• Пример экспорта перечислимого типа и структуры в Python.
• Использование тегов py class и py struct для экспорта Rust-структур в Python.
• Пример с электронной почтой и вложениями.

07:57 Публикация модуля и Rust Import

• Публикация модуля Python на основе Rust в PyPI с помощью Maturin.
• Альтернатива Maturin - Rust Import для быстрого импорта Rust-кода в Python.
• Rust Import подходит для прототипов и быстрого импорта Rust-кода.

10:23 Использование Rust в Python

• Автор спрашивает, пользовался ли кто-то из зрителей инструментами Rust.
• Рассматривает возможность более широкого использования Rust в скриптах на Python.
• Делится своим опытом работы с Rust и разработкой нескольких инструментов.

10:38 Планы на будущее и рекомендации

• Автор планирует сделать еще одно видео о Rust, где покажет, как он это сделал и какие уроки извлек.
• Считает Rust приятным языком программирования, но не думает, что он заменит Python.
• Рекомендует посмотреть следующее [[Введение в программирование на Rust для любителей Python|видео]] для более глубокого погружения в Rust.
• Благодарит за просмотр и прощается до следующего раза.

