---
tags:
  - python
  - SQL
  - excel
---

Для выгрузки результатов запросов из MS SQL Server в Excel на отдельные листы с использованием Python можно воспользоваться библиотеками `pyodbc` (для работы с SQL Server) и `pandas` (для обработки данных и записи в Excel). Вот пошаговое решение:

---

### 1. Установите необходимые библиотеки
```bash
pip install pyodbc pandas openpyxl
```

---

### 2. Пример кода
```python
import pyodbc
import pandas as pd

# Настройки подключения к SQL Server
server = "ваш_сервер"
database = "ваша_база"
username = "ваш_логин"
password = "ваш_пароль"

# Создайте строку подключения
conn_str = (
    f"DRIVER={{ODBC Driver 17 for SQL Server}};"
    f"SERVER={server};"
    f"DATABASE={database};"
    f"UID={username};"
    f"PWD={password};"
)

# Список запросов и названий листов в Excel
queries = {
    "Лист1": "SELECT * FROM таблица1 WHERE условие1",
    "Лист2": "SELECT * FROM таблица2 WHERE условие2",
    # Добавьте свои запросы
}

# Подключение к SQL Server
try:
    conn = pyodbc.connect(conn_str)
    print("Успешное подключение к SQL Server")
except Exception as e:
    print(f"Ошибка подключения: {e}")
    exit()

# Создайте Excel-файл
excel_file = "результаты.xlsx"
with pd.ExcelWriter(excel_file, engine='openpyxl') as writer:
    for sheet_name, query in querries.items():
        try:
            # Выполните запрос и получите данные
            df = pd.read_sql(query, conn)
            # Запишите данные в лист Excel
            df.to_excel(writer, sheet_name=sheet_name, index=False)
            print(f"Лист '{sheet_name}' создан")
        except Exception as e:
            print(f"Ошибка в запросе '{sheet_name}': {e}")

# Закройте соединение
conn.close()
print(f"Данные сохранены в файл: {excel_file}")
```

---

### 3. Пояснения
1. **Подключение к SQL Server**: Используется строка подключения с указанием драйвера, сервера, базы данных и учетных данных.
2. **Структура запросов**: Запросы хранятся в словаре, где ключи — названия листов в Excel, а значения — SQL-запросы.
3. **Запись в Excel**: 
   - Данные из каждого запроса сохраняются в отдельный лист с помощью `pd.ExcelWriter`.
   - Параметр `engine='openpyxl'` позволяет работать с форматом `.xlsx`.
4. **Обработка ошибок**: Добавлена базовая обработка исключений для подключения и выполнения запросов.

---

### 4. Важные замечания
- Убедитесь, что ODBC Driver for SQL Server установлен (можно скачать [здесь](https://learn.microsoft.com/ru-ru/sql/connect/odbc/download-odbc-driver-for-sql-server)).
- Названия листов должны быть короче 31 символа и не содержать запрещенных символов (`:`, `\`, `?`, `*`, `[`, `]`).
- Для больших объемов данных используйте `chunksize` в `pd.read_sql`.

---

Это решение позволяет гибко управлять запросами и автоматизировать выгрузку данных в структурированный Excel-файл.