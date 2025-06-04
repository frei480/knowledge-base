---
tags:
  - SQL
---

Для решения задачи необходимо выбрать документы, у которых разница между датами регистрации входящего и исходящего номеров превышает 20 дней. Предполагается, что таблица содержит поля `RegistrationDateIn` (дата регистрации входящего документа) и `RegistrationDateOut` (дата регистрации исходящего документа).

### SQL-запрос:
```sql
SELECT *
FROM Documents
WHERE 
    RegistrationDateIn IS NOT NULL
    AND RegistrationDateOut IS NOT NULL
    AND ABS(DATEDIFF(day, RegistrationDateIn, RegistrationDateOut)) > 20;
```

### Пояснение:
1. **Проверка на NULL**: Исключаются записи, где одна из дат не указана.
2. **Вычисление разницы в днях**: 
   - `DATEDIFF(day, StartDate, EndDate)` возвращает разницу в днях между `StartDate` и `EndDate`.
   - `ABS()` используется для учета разницы в любом направлении (если важно, чтобы именно одна дата была позже другой, уберите `ABS` и укажите порядок дат явно).
3. **Условие `> 20`**: Отбирает документы с разницей более 20 дней.

### Вариант без ABS (если исходящая дата должна быть позже входящей):
```sql
SELECT *
FROM Documents
WHERE 
    RegistrationDateIn IS NOT NULL
    AND RegistrationDateOut IS NOT NULL
    AND DATEDIFF(day, RegistrationDateIn, RegistrationDateOut) > 20;
```

Этот запрос вернет документы, где исходящая дата регистрации превышает входящую более чем на 20 дней.