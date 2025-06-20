---
tags:
  - SQL
---

документ при создании сразу получает ревизию A.1, а каждое изменение увеличивает номер (A.2, A.3 и т.д.), то решение будет таким:

### Структура таблиц (с добавлением даты создания документа)
```sql
CREATE TABLE documents (
  id INT PRIMARY KEY,
  title TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL -- Важно для определения исходной версии
);

CREATE TABLE changes (
  id INT PRIMARY KEY,
  document_id INT REFERENCES documents(id),
  changed_at TIMESTAMP NOT NULL, -- Дата изменения
  details TEXT NOT NULL
);
```

### Запрос для генерации ревизий (включая исходную A.1)
```sql
WITH document_versions AS (
  -- Исходная версия документа (A.1)
  SELECT 
    id AS document_id,
    title,
    NULL AS change_id,
    'Initial document' AS details,
    created_at AS version_date,
    1 AS version_order
  FROM documents
  
  UNION ALL
  
  -- Все изменения документа (A.2, A.3, ...)
  SELECT
    d.id,
    d.title,
    c.id AS change_id,
    c.details,
    c.changed_at,
    2 AS version_order -- Для правильной сортировки относительно исходной
  FROM documents d
  JOIN changes c ON d.id = c.document_id
)

SELECT
  document_id,
  title,
  change_id,
  details,
  'A.' || ROW_NUMBER() OVER (
    PARTITION BY document_id 
    ORDER BY version_date, version_order, change_id
  ) AS revision
FROM document_versions
ORDER BY document_id, revision;
```

### Пояснения:
1. **Исходная версия (A.1)**  
   - Берется прямо из таблицы `documents`
   - `version_order = 1` гарантирует приоритет над изменениями при одинаковой дате
   - `change_id = NULL` (так как это не изменение)
   - Фиксированный текст `'Initial document'`

2. **Изменения (A.2+)**
   - `version_order = 2` для сортировки после исходной версии
   - Порядок определяется:
     * `version_date` (основная сортировка по времени)
     * `version_order` (гарантирует A.1 → A.2)
     * `change_id` (разрешает коллизии при одинаковых датах)

3. **Формирование ревизии**  
   `'A.' || ROW_NUMBER()` создает последовательность:
   - Для нового документа: A.1
   - После 1 изменения: A.1 → A.2
   - После 2 изменений: A.1 → A.2 → A.3

### Пример результата
| document_id | title     | change_id | details              | revision |
|-------------|-----------|-----------|----------------------|----------|
| 1           | Contract  | NULL      | Initial document     | A.1      |
| 1           | Contract  | 55        | Updated clause 4.2   | A.2      |
| 1           | Contract  | 78        | Added annex          | A.3      |
| 2           | Report    | NULL      | Initial document     | A.1      |
| 3           | Agreement | NULL      | Initial document     | A.1      |
| 3           | Agreement | 92        | Fixed contacts       | A.2      |

### Особенности реализации:
1. **Документы без изменений**  
   Отображаются как одна строка с ревизией A.1
   
2. **Хронологический порядок**  
   Ревизии всегда формируются в порядке:
   - Дата создания документа
   - Дата первого изменения
   - Дата второго изменения и т.д.

3. **Гарантия последовательности**  
   Даже если изменения были добавлены в БД "вне очереди", сортировка по `version_date` + `change_id` обеспечит правильную нумерацию.

4. **Производительность**  
   Для больших наборов данных рекомендуется добавить индекс:
   ```sql
   CREATE INDEX idx_changes_document ON changes(document_id, changed_at);
   ```

Это решение гарантирует, что каждый документ начнется с A.1, а все изменения будут последовательно пронумерованы в историческом порядке.