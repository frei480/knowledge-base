---
tags:
  - SQL
  - python
---

Чтобы преобразовать строку из таблицы MS SQL в словарь Python с использованием драйвера `pyodbc`, выполните следующие шаги:

# 1. Подключение К Базе Данных
```python
import pyodbc

# Настройки подключения
server = 'ваш_сервер'
database = 'ваша_база'
username = 'ваш_логин'
password = 'ваш_пароль'

# Создание подключения
conn = pyodbc.connect(
    f'DRIVER={{ODBC Driver 17 for SQL Server}};'
    f'SERVER={server};'
    f'DATABASE={database};'
    f'UID={username};'
    f'PWD={password};'
)
cursor = conn.cursor()
```

# 2. Выполнение SQL-запроса
```python
query = "SELECT * FROM ваша_таблица"
cursor.execute(query)
```

# 3. Преобразование Результатов В Словарь
Используйте `cursor.description` для получения названий колонок и объедините их со значениями строк через `zip()`:
```python
# Получение названий колонок
columns = [column[0] for column in cursor.description]

# Преобразование всех строк в список словарей
result = []
for row in cursor.fetchall():
    row_dict = dict(zip(columns, row))
    result.append(row_dict)

# Либо в виде спискового включения
result = [dict(zip(columns, row)) for row in cursor.fetchall()]
```

# 4. Закрытие Подключения
```python
cursor.close()
conn.close()
```

# Пример Итогового Кода
```python
import pyodbc

def get_data_as_dict():
    conn = pyodbc.connect(
        'DRIVER={ODBC Driver 17 for SQL Server};'
        'SERVER=ваш_сервер;'
        'DATABASE=ваша_база;'
        'UID=ваш_логин;'
        'PWD=ваш_пароль;'
    )
    cursor = conn.cursor()
    
    cursor.execute("SELECT * FROM ваша_таблица")
    columns = [col[0] for col in cursor.description]
    result = [dict(zip(columns, row)) for row in cursor.fetchall()]
    
    cursor.close()
    conn.close()
    return result

# Использование
data = get_data_as_dict()
print(data[0])  # Вывод первой строки в виде словаря
```

# Примечания:
1. **Уникальность имен колонок**: Убедитесь, что в SQL-запросе нет дубликатов имен колонок (например, при использовании `JOIN`).
2. **Типы данных**: PyODBC автоматически конвертирует данные SQL в Python-типы.
3. **Производительность**: Для больших объемов данных используйте `cursor.fetchmany(size)`.

Этот подход позволяет легко работать с результатами SQL-запросов как со словарями, что удобно для обработки данных в Python.