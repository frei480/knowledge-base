---
tags:
  - SQL
---

**GROUP BY** и **PARTITION BY** в SQL используются для группировки данных, но решают разные задачи и работают в разных контекстах. Вот подробное объяснение:

---

# **1. GROUP BY**
Используется для **агрегации данных** (суммирование, подсчет, среднее и т.д.) по группам.
**Особенности**:
- Объединяет строки с одинаковыми значениями в группы.
- **Уменьшает количество строк** в результате (одна строка на группу).
- Все неагрегированные колонки в `SELECT` должны быть указаны в `GROUP BY`.

**Пример**:
```sql
SELECT department, SUM(salary) AS total_salary
FROM employees
GROUP BY department;
```
Результат:
| department | total_salary |
|------------|--------------|
| IT         | 50000        |
| HR         | 30000        |

---

# **2. PARTITION BY**
Используется в **оконных функциях** (`OVER()`), чтобы разделить данные на группы (**секции**) для вычислений **без объединения строк**.
**Особенности**:
- Не уменьшает количество строк (все строки сохраняются).
- Позволяет выполнять агрегацию или ранжирование внутри секций.
- Часто используется с функциями: `ROW_NUMBER()`, `RANK()`, `SUM()`, `AVG()` и др.

**Пример**:
```sql
SELECT name, department, salary,
       SUM(salary) OVER (PARTITION BY department) AS dept_total_salary
FROM employees;
```
Результат (сохраняются все строки):
| name  | department | salary | dept_total_salary |
|-------|------------|--------|--------------------|
| Alice | IT         | 30000  | 50000             |
| Bob   | IT         | 20000  | 50000             |
| Eve   | HR         | 30000  | 30000             |

---

# **Ключевые различия**
| Характеристика       | GROUP BY                          | PARTITION BY                     |
|----------------------|-----------------------------------|-----------------------------------|
| **Изменение строк**  | Сворачивает группы в одну строку  | Сохраняет все строки             |
| **Контекст**         | Используется с агрегатными функциями (SUM, COUNT) | Используется с оконными функциями (OVER()) |
| **Результат**        | Агрегированные данные             | Детальные данные + вычисления    |

---

# **Примеры использования**
**С GROUP BY**:
```sql
-- Средняя зарплата по отделам
SELECT department, AVG(salary) 
FROM employees 
GROUP BY department;
```

**С PARTITION BY**:
```sql
-- Ранжирование сотрудников по зарплате внутри отдела
SELECT name, department, salary,
       RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
FROM employees;
```

---

# **Когда Что использовать?**
- **GROUP BY**: Когда нужны итоговые данные по группам (например, сумма продаж за месяц).
- **PARTITION BY**: Когда нужны вычисления внутри групп без потери деталей (например, доля зарплаты сотрудника в отделе).

Если нужно объединить оба подхода, можно использовать **подзапросы** или **CTE**. Например:
```sql
WITH department_totals AS (
    SELECT department, SUM(salary) AS total
    FROM employees
    GROUP BY department
)
SELECT e.name, e.department, e.salary,
       e.salary / dt.total * 100 AS salary_percent
FROM employees e
JOIN department_totals dt ON e.department = dt.department;
```