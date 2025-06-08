---
tags:
  - python
---

Чтобы распараллелить вычисление суммы ряда в Python, можно использовать модуль `multiprocessing`. Разделим диапазон на части и вычислим каждую часть в отдельном процессе. Вот реализация:

```python
import multiprocessing

def partial_sum(start, end):
    s = 0
    for i in range(start, end + 1):
        s += 1 / i
    return s

def main():
    start = 2
    end = 99999  # т.к. range(2, 100000) включает числа до 99999
    n_processes = multiprocessing.cpu_count()  # Используем все доступные ядра

    # Разделяем диапазон на поддиапазоны
    chunk_size = (end - start + 1) // n_processes
    ranges = []
    for i in range(n_processes):
        s = start + i * chunk_size
        e = s + chunk_size - 1
        if i == n_processes - 1:
            e = end  # Последний диапазон покрывает остаток
        ranges.append((s, e))

    # Создаем пул процессов и вычисляем
    with multiprocessing.Pool() as pool:
        results = pool.starmap(partial_sum, ranges)
        total = sum(results)
        print(f"Result: {total}")

if __name__ == '__main__':
    main()
```

### Пояснения:
1. **Разделение диапазона**: Диапазон `2–99999` делится на равные части по количеству ядер CPU.
2. **Функция `partial_sum`**: Вычисляет сумму `1/i` для своего поддиапазона.
3. **Пул процессов**: Задачи распределяются между процессами, результаты суммируются.

### Оптимизации:
- Используйте `n_processes = 4` (или другое число) вручную, если нужно ограничить ресурсы.
- Для очень больших `end` можно увеличить `n_processes`, но учитывайте накладные расходы.