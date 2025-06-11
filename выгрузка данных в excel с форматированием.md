---
tags:
  - excel
  - SQL
  - python
---

Чтобы экспортировать данные из pandas в Excel и покрасить определённые столбцы в красный цвет, используйте комбинацию **pandas** и **openpyxl**. Вот пошаговое решение:

# 1. Установите Необходимые Библиотеки
```bash
pip install pandas openpyxl
```

# 2. Пример Кода
```python
import pandas as pd
from openpyxl import load_workbook
from openpyxl.styles import PatternFill

# Создаем пример DataFrame
data = {
    'Name': ['Alice', 'Bob', 'Charlie'],
    'Age': [25, 30, 35],
    'Score': [85, 92, 78]
}
df = pd.DataFrame(data)

# Сохраняем DataFrame в Excel без форматирования
excel_file = "output.xlsx"
df.to_excel(excel_file, index=False, engine='openpyxl')

# Загружаем книгу Excel для форматирования
wb = load_workbook(excel_file)
ws = wb.active

# Создаем стиль заливки (красный цвет)
red_fill = PatternFill(start_color='FFFF0000', end_color='FFFF0000', fill_type='solid')

# Указываем столбцы для покраски (например: 'Age' и 'Score')
columns_to_color = ['Age', 'Score']

# Находим номера столбцов по их названиям
column_indices = {}
for idx, cell in enumerate(ws[1], 1):  # Первая строка - заголовки
    if cell.value in columns_to_color:
        column_indices[cell.value] = idx

# Применяем заливку ко всем ячейкам в выбранных столбцах (кроме заголовка)
for col_name, col_idx in column_indices.items():
    for row in range(2, ws.max_row + 1):  # Со 2-й строки до конца
        ws.cell(row=row, column=col_idx).fill = red_fill

# Сохраняем изменения
wb.save(excel_file)
```

# Как Это Работает:
1. **Экспорт данных**: Сначала данные сохраняются в стандартный Excel-файл.
2. **Форматирование**:
   - Открываем файл с помощью `openpyxl`.
   - Определяем столбцы для форматирования по их названиям в первой строке.
   - Закрашиваем все ячейки в указанных столбцах (кроме заголовка) в красный цвет.

# Для Динамического Форматирования:
Если нужно красить столбцы по условию (например, значения < 30), замените цикл на:
```python
for col_name, col_idx in column_indices.items():
    for row in range(2, ws.max_row + 1):
        cell = ws.cell(row=row, column=col_idx)
        # Условие: покрасить если значение < 30
        if isinstance(cell.value, (int, float)) and cell.value < 30:
            cell.fill = red_fill
```

# Результат:
| Name    | Age (красный) | Score (красный) |
|---------|---------------|-----------------|
| Alice   | 25            | 85              |
| Bob     | 30            | 92              |
| Charlie | 35            | 78              |

> **Важно:**
> - Используйте `engine='openpyxl'` при сохранении DataFrame.
> - Цвета задаются в HEX-формате (красный: `FFFF0000`).
> - Для работы с условиями добавьте проверки типов данных (числа, строки и т.д.).

Этот подход гарантирует полный контроль над стилизацией ячеек Excel после экспорта данных из pandas.