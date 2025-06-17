---
tags:
  - python
---
# Противодействие SQL-инъекциям В Python
SQL-инъекции — это атаки, при которых злоумышленник внедряет вредоносный SQL-код через входные данные приложения. Это может привести к:
- Утечке/изменению данных
- Удалению таблиц
- Получению контроля над СУБД

---

## Основные Методы Защиты
1. **Параметризованные запросы**
   Используйте placeholders (`?`, `%s`, `:name`) вместо конкатенации строк.

2. **ORM (Object-Relational Mapping)**
   Автоматическое экранирование параметров через фреймворки вроде SQLAlchemy.

3. **Экранирование входных данных**
   Используйте встроенные функции экранирования (не как основной метод!).

4. **Валидация входных данных**
   Проверяйте тип, длину и формат данных.

5. **Принцип минимальных привилегий**
   Ограничьте права пользователя БД только необходимыми операциями.

---

# Примеры Кода

## 1. Параметризованные Запросы
**Уязвимый код (конкатенация строк):**
```python
import sqlite3

username = input("Username: ")  # Ввод: ' OR 1=1 --
conn = sqlite3.connect('test.db')
cursor = conn.cursor()
cursor.execute("SELECT * FROM users WHERE username = '" + username + "'")  # Риск инъекции!
print(cursor.fetchall())
```

**Исправленная версия (с параметрами):**
```python
username = input("Username: ")
conn = sqlite3.connect('test.db')
cursor = conn.cursor()
cursor.execute("SELECT * FROM users WHERE username = ?", (username,))  # Безопасно
print(cursor.fetchall())
```

**Для PostgreSQL (psycopg2):**
```python
cursor.execute("SELECT * FROM users WHERE username = %s", (username,))
```

**Для MySQL (mysql-connector):**
```python
cursor.execute("SELECT * FROM users WHERE username = %s", (username,))
```

---

## 2. Использование ORM (SQLAlchemy)
```python
from sqlalchemy import create_engine, text
from sqlalchemy.orm import sessionmaker

engine = create_engine('sqlite:///test.db')
Session = sessionmaker(bind=engine)
session = Session()

username = input("Username: ")
# Безопасный запрос через text() с параметрами
query = text("SELECT * FROM users WHERE username = :username")
result = session.execute(query, {"username": username})
print(result.fetchall())
```

---

## 3. Экранирование Данных (второстепенный метод)
Используйте только если параметризация невозможна:
```python
from psycopg2 import extensions  # Для PostgreSQL

username = extensions.adapt(input("Username: ")).getquoted().decode()
# Теперь username экранирован, но лучше использовать параметризованные запросы!
```

---

## 4. Валидация Входных Данных
```python
import re

def validate_username(username: str) -> bool:
    # Проверка: только буквы/цифры, длина 4-20 символов
    return bool(re.match(r'^[a-zA-Z0-9]{4,20}$', username))

username = input("Username: ")
if not validate_username(username):
    raise ValueError("Invalid username")
```

---

# Дополнительные Меры Защиты
- **Stored Procedures:** Используйте их с параметрами.
- **Web Application Firewall (WAF):** Фильтруйте подозрительные запросы.
- **Регулярные аудиты:** Проверяйте код и логи БД.
- **Лимит прав:** Запретите DROP/DELETE для пользователя приложения.

---

# Тест На Уязвимость
Попробуйте ввести в ваши формы:
```sql
' OR '1'='1
'; DROP TABLE users; --
```
Если приложение возвращает ошибку SQL или выполняет команду — оно уязвимо!

---

**Итог:** Всегда используйте параметризованные запросы или ORM. Никогда не доверяйте пользовательскому вводу без валидации.