---
tags:
  - SQL
---
Чтобы оптимизировать SQL-запрос с `JOIN` для больших таблиц, сначала отфильтруйте данные из первой таблицы с помощью `WHERE`, а затем выполните соединение. Это можно сделать через **подзапрос** или **CTE**, чтобы явно указать порядок операций. Вот как это реализовать:

### 1. Использование подзапроса
```sql
SELECT *
FROM (
    SELECT * 
    FROM table1 
    WHERE condition = 'value'  -- Сначала фильтруем первую таблицу
) AS filtered_table1
JOIN table2 
    ON filtered_table1.id = table2.id;  -- Затем выполняем JOIN
```

### 2. Через CTE (Common Table Expression) [[SQL Common Table Expression]]
```sql
WITH filtered_table1 AS (
    SELECT * 
    FROM table1 
    WHERE condition = 'value'  -- Фильтрация на первом шаге
)
SELECT *
FROM filtered_table1
JOIN table2 
    ON filtered_table1.id = table2.id;  -- Соединение после фильтрации
```

### Почему это работает?
- **Фильтрация до JOIN**: Сначала сокращается объем данных из первой таблицы, что уменьшает нагрузку на операцию соединения.
- **Использование индексов**: Если в `table1` есть индекс для колонки в условии `WHERE`, фильтрация выполнится очень быстро.
- **Порядок выполнения**: SQL обычно выполняет подзапросы/CTE перед основным запросом, что гарантирует оптимизацию.

### Дополнительные советы:
- Добавьте **индексы** на колонки, участвующие в `WHERE` и `JOIN`.
- Используйте `EXPLAIN` для анализа плана выполнения и убедитесь, что оптимизатор следует ожидаемому порядку.
- Для `LEFT JOIN` переносите условия из `WHERE` в `ON`, если нужно сохранить все строки из первой таблицы:
  ```sql
  SELECT *
  FROM table1
  LEFT JOIN table2 
      ON table1.id = table2.id 
      AND table1.condition = 'value';  -- Условие в ON, а не WHERE
  ```

Пример с фильтрацией и `LEFT JOIN`:
```sql
SELECT *
FROM (
    SELECT * 
    FROM table1 
    WHERE condition = 'value'
) AS filtered_table1
LEFT JOIN table2 
    ON filtered_table1.id = table2.id;
```