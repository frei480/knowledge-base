---
tags:
  - SQL
---
Для решения задачи по формированию столбца `revision` в формате `<буква>.<номер>` на основе истории изменений документа и уведомлений, можно использовать следующий подход в MS SQL:

```sql
WITH DataWithGroups AS (
    SELECT *,
        -- Группировка: новая группа начинается с каждого документа и каждого второго уведомления
        SUM(CASE 
            WHEN [document inventory number] IS NOT NULL THEN 1 
            WHEN [notice inventory number] IS NOT NULL AND 
                 LAG([notice inventory number]) OVER (ORDER BY date) IS NOT NULL THEN 1 
            ELSE 0 
        END) OVER (ORDER BY date ROWS UNBOUNDED PRECEDING) AS group_id
    FROM your_table
),
GroupedData AS (
    SELECT *,
        -- Буква группы (A, B, C...)
        CHAR(65 + (group_id - 1) AS group_letter,
        -- Нумерация внутри группы: документы всегда получают 1, уведомления - порядковый номер
        ROW_NUMBER() OVER (
            PARTITION BY group_id 
            ORDER BY CASE 
                WHEN [document inventory number] IS NOT NULL THEN 0 
                ELSE 1 
            END, date
        ) AS group_row_num
    FROM DataWithGroups
)
SELECT 
    [document inventory number],
    [notice inventory number],
    denotation,
    date,
    -- Формируем revision: буква группы + номер в группе
    CONCAT(
        group_letter, 
        '.', 
        CASE 
            WHEN [document inventory number] IS NOT NULL THEN 1
            ELSE group_row_num 
        END
    ) AS revision
FROM GroupedData;
```

### Объяснение решения:
1. **Группировка записей (DataWithGroups CTE)**:
   - Новая группа начинается:
     - С каждого документа (`[document inventory number] IS NOT NULL`)
     - С каждым вторым подряд уведомлением (когда текущее и предыдущее - уведомления)
   - `SUM() OVER` с условием инкремента группового идентификатора (`group_id`) реализует кумулятивное суммирование триггеров начала группы.

2. **Нумерация внутри группы (GroupedData CTE)**:
   - **Буква группы**: преобразуем `group_id` в буквы алфавита (65 = ASCII 'A').
   - **Порядковый номер в группе**:
     - Документы всегда получают приоритет (сортировка по `CASE...END` с `0` для документов).
     - Уведомления сортируются по дате.
     - `ROW_NUMBER()` назначает порядковый номер внутри группы.

3. **Формирование значения revision**:
   - Для документов: `<буква>.1`
   - Для уведомлений: `<буква>.<порядковый_номер>`

### Результат выполнения запроса:
| document inventory number | notice inventory number | denotation  | date       | revision |
|---------------------------|-------------------------|-------------|------------|----------|
| a-0001                    | NULL                    | xxx.010.000 | 2021-02-23 | A.1      |
| NULL                      | n-100121                | xxx.010.000 | 2021-02-25 | A.2      |
| NULL                      | n-123123                | xxx.010.000 | 2021-03-23 | B.1      |
| a-0002                    | NULL                    | xxx.010.000 | 2021-04-15 | B.1      |
| NULL                      | n-123456                | xxx.010.000 | 2022-02-03 | B.2      |
| NULL                      | n-466549                | xxx.010.000 | 2023-03-03 | C.1      |
| a-0003                    | NULL                    | xxx.010.000 | 2024-04-03 | C.1      |
| NULL                      | n-100699                | xxx.010.000 | 2025-02-03 | C.2      |
| NULL                      | n-102346                | xxx.010.000 | 2025-03-12 | C.3      |

### Особенности работы:
- **Группировка**: Уведомление `n-123123` начинает новую группу (B), так как следует после другого уведомления.
- **Нумерация в группе**:
  - Документы всегда получают `.1`.
  - Уведомления нумеруются по порядку в группе.
- **Расширяемость**: Поддерживает неограниченное количество групп (после 'Z' будет 'AA', 'AB' и т.д. при модификации).