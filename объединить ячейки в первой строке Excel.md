---
tags:
  - excel
  - python
---
Чтобы объединить ячейки в первой строке Excel при экспорте из pandas и добавить туда текст, выполните следующие шаги:

# Основные Шаги:
1. **Используйте `pd.ExcelWriter` с движком `openpyxl`**
2. **Запишите DataFrame начиная со второй строки** (чтобы оставить место для заголовка)
3. **Вручную объедините ячейки и добавьте текст** через объект листа

# Пример Кода:
```python
import pandas as pd
from openpyxl import Workbook
from openpyxl.utils.dataframe import dataframe_to_rows

# Пример DataFrame
data = {'Column1': [1, 2, 3], 'Column2': ['A', 'B', 'C']}
df = pd.DataFrame(data)

# Создаем Excel-файл
with pd.ExcelWriter('output.xlsx', engine='openpyxl') as writer:
    # Записываем DataFrame, начиная со 2-й строки (row=2)
    df.to_excel(writer, sheet_name='Sheet1', startrow=1, index=False)
    
    # Получаем объекты книги и листа
    workbook = writer.book
    worksheet = writer.sheets['Sheet1']
    
    # Объединяем ячейки A1:B1 (или другие нужные)
    worksheet.merge_cells('A1:B1')
    
    # Добавляем текст в объединенную ячейку
    worksheet['A1'] = "Ваш заголовок здесь"
    
    # Опционально: стилизация
    worksheet['A1'].font = openpyxl.styles.Font(bold=True, size=14)
    worksheet['A1'].alignment = openpyxl.styles.Alignment(horizontal='center', vertical='center')

# Файл автоматически сохранится при выходе из блока with
```

# Пояснения:
1. **`startrow=1`**
   Сдвигает начало DataFrame на вторую строку (индексация с 0).

2. **`merge_cells('A1:B1')`**
   Объединяет ячейки от A1 до B1. Измените диапазон под ваши нужды (например, `'A1:D1'`).

3. **Стилизация** (опционально):
   - `font` - жирный шрифт, размер
   - `alignment` - выравнивание по центру
   - `fill` - заливка цветом (используйте `PatternFill`)

# Для Объединения Всех Колонок:
Если нужно объединить ячейки во всей первой строке:
```python
# Определите последнюю колонку
last_col = chr(65 + len(df.columns) - 1)  # 65 = 'A'
worksheet.merge_cells(f'A1:{last_col}1')
```

# Требования:
```bash
pip install pandas openpyxl
```

Этот метод сохраняет все функциональности pandas для экспорта данных, добавляя возможность кастомного форматирования через openpyxl.