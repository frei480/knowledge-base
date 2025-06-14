---
tags:
  - SQL
---
Использование `GROUP BY` с константными значениями, например `GROUP BY 1, 2`, — это **указание порядковых номеров столбцов** из списка `SELECT`, а не их явных имён или выражений. Такой подход работает во многих СУБД (например, PostgreSQL, MySQL), но **не соответствует стандарту SQL** и имеет ряд нюансов.

---

### Как это работает?
- Цифры `1, 2, ...` в `GROUP BY` означают порядковые номера столбцов, перечисленных в `SELECT`.  
- Например:
  ```sql
  SELECT 
      name,         -- столбец 1
      age,          -- столбец 2
      COUNT(*) 
  FROM users 
  GROUP BY 1, 2;    -- эквивалентно GROUP BY name, age
  ```
  Здесь `GROUP BY 1, 2` группирует данные по первому и второму столбцам из `SELECT` (т.е. по `name` и `age`).

---

### Плюсы и минусы
#### ✅ **Плюсы**:
- Удобно для быстрого написания запросов, особенно если столбцы в `SELECT` имеют сложные названия или выражения.
- Сокращает код, если в `GROUP BY` нужно указать много столбцов.

#### ❌ **Минусы**:
1. **Непрозрачность**:
   - Запрос становится менее читаемым. Номера столбцов неочевидны, особенно для новичков.
2. **Хрупкость**:
   - Если изменить порядок или количество столбцов в `SELECT`, логика `GROUP BY` сломается.  
   Например:
   ```sql
   -- Исходный запрос:
   SELECT age, name, COUNT(*) FROM users GROUP BY 1, 2; -- группировка по age, name

   -- Если поменять порядок в SELECT:
   SELECT name, age, COUNT(*) FROM users GROUP BY 1, 2; -- группировка по name, age (логика изменилась!).
   ```
3. **Несовместимость со стандартом SQL**:
   - В стандарте SQL требуется указывать явные имена столбцов или выражения в `GROUP BY`. Использование чисел поддерживается не всеми СУБД (например, **Oracle** и **SQL Server** такое не допускают).

---

### Примеры
#### 1. Простая группировка:
```sql
SELECT 
    department,   -- столбец 1
    salary,       -- столбец 2
    COUNT(*) 
FROM employees 
GROUP BY 1, 2;    -- группировка по department и salary
```

#### 2. Группировка с алиасами:
```sql
SELECT 
    UPPER(name) AS upper_name,  -- столбец 1
    age * 2 AS double_age       -- столбец 2
FROM users 
GROUP BY 1, 2;                  -- группировка по upper_name и double_age
```

---

### Рекомендации
1. **Избегайте использования чисел в `GROUP BY`**:
   - Всегда указывайте явные имена столбцов или выражения. Это делает код:
     - Читаемым.
     - Устойчивым к изменениям в `SELECT`.
     - Совместимым со стандартом SQL.
   ```sql
   -- Лучше:
   SELECT department, salary, COUNT(*) 
   FROM employees 
   GROUP BY department, salary;
   ```

2. **Исключения**:
   - Можно использовать числа для **быстрого прототипирования** или в случаях, где порядок столбцов гарантированно не изменится (например, в скриптах для личного использования).

---

### Важно!
Если вы всё же используете `GROUP BY` с числами:
- Убедитесь, что порядок столбцов в `SELECT` не поменяется.
- Документируйте запрос, чтобы другие разработчики понимали логику.