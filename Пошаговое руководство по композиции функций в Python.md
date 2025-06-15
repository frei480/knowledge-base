---
tags:
  - python
---

**Пошаговое руководство по композиции функций в Python**
_С примерами и практическими применениями_

---
# **1. Что Такое Композиция функций?**
Композиция функций — это объединение нескольких функций в цепочку, где результат одной функции передается в качестве аргумента следующей.
**Математическая аналогия**: `h(x) = f(g(x))` → `h = f ∘ g`.

---
# **2. Зачем Это нужно?**
- Упрощает сложные операции, разбивая их на простые шаги.
- Повышает читаемость кода.
- Позволяет переиспользовать функции.
---
# **3. Базовый Пример: Создание функции-композитора**

Реализуем функцию `compose`, которая объединяет функции в порядке их выполнения (справа налево).

```python
def compose(*funcs):
    def inner(arg):
        result = arg
        for func in reversed(funcs):
            result = func(result)
        return result
    return inner
```

**Пример использования**:
```python
def add_5(x):
    return x + 5

def multiply_3(x):
    return x * 3

composed = compose(add_5, multiply_3)  # Сначала multiply_3, потом add_5
print(composed(4))  # (4 * 3) + 5 = 17
```
---
# **4. Примеры композиции**

## **Пример 1: Обработка строк**
```python
def to_upper(s):
    return s.upper()

def add_exclamation(s):
    return s + "!"

process_text = compose(add_exclamation, to_upper)
print(process_text("hello"))  # "HELLO!"
```

## **Пример 2: Фильтрация данных**
```python
def filter_even(nums):
    return [x for x in nums if x % 2 == 0]

def sum_list(nums):
    return sum(nums)

pipeline = compose(sum_list, filter_even)
print(pipeline([1, 2, 3, 4, 5]))  # Сумма четных чисел: 2 + 4 = 6
```

## **Пример 3: Конвейер преобразований**
```python
def square(x):
    return x ** 2

def half(x):
    return x / 2

composed = compose(half, square)  # Сначала возводим в квадрат, затем делим на 2
print(composed(6))  # (6^2)/2 = 18
```
---
# **5. Композиция С Помощью `functools.reduce`**

Для более гибкой композиции можно использовать стандартную библиотеку:
```python
from functools import reduce

def compose(*funcs):
    return lambda x: reduce(lambda acc, f: f(acc), reversed(funcs), x)
```

**Пример**:
```python
composed = compose(str, lambda x: x + 10, lambda x: x * 2)
print(composed(5))  # (5 * 2) + 10 = 20 → "20"
```
---
# **6. Композиция С лямбда-функциями**
Можно создавать цепочки на лету:
```python
pipeline = compose(
    lambda x: x * 10,
    lambda x: x - 5,
    lambda x: x + 2
)
print(pipeline(3))  # ((3 + 2) - 5) * 10 = 0
```
---
# **7. Практическое Применение: Обработка данных**
**Задача**: Преобразовать список чисел: отфильтровать отрицательные, взять квадраты, округлить до целых.
```python
def filter_negatives(nums):
    return [x for x in nums if x >= 0]

def square_all(nums):
    return [x**2 for x in nums]

def round_numbers(nums):
    return [round(x) for x in nums]

pipeline = compose(round_numbers, square_all, filter_negatives)
data = [-3, 1.2, 4, 5.7, -10]
print(pipeline(data))  # [1, 16, 32]
```

---

# **8. Советы И предупреждения**

1. **Порядок функций**: `compose(f, g)(x)` → `f(g(x))`.
2. **Читаемость**: Не создавайте слишком длинные цепочки.
3. **Типы данных**: Убедитесь, что выход одного типа совместим со входом следующей функции.
---
# **9. Альтернативные подходы**
- **Библиотеки**: Используйте `toolz` (функция `compose`) или `numpy` для векторных операций.
- **Классы**: Реализуйте цепочки методов через объекты (паттерн **Builder**).
---

# **Заключение**
Композиция функций делает код модульным и выразительным. Начните с простых примеров, постепенно переходя к сложным конвейерам обработки данных.