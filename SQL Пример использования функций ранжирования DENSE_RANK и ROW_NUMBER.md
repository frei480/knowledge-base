---
tags:
  - SQL
---
# Присвоить Наименование Группе Записей
В SQL Server вы можете вывести записи, сгруппированные по определенному столбцу, и каждой группе присвоить последовательное имя (a, b, c и так далее). Для этого используются оконные функции, в частности DENSE_RANK(), и затем преобразование числа в букву.

Вот как это можно сделать:

Допустим, у вас есть таблица YourTable со столбцами GroupName (по которому нужно группировать) и Value (или другими столбцами, которые вы хотите вывести для каждой записи).
```sql
SELECT  
    CHAR(96 + DENSE_RANK() OVER (ORDER BY GroupName)) AS GroupLetter,  
    T.GroupName,  
    T.Value  
FROM  
    YourTable AS T  
ORDER BY  
    GroupLetter, T.GroupName, T.Value;  
```
---
Объяснение:
1. `DENSE_RANK() OVER (ORDER BY GroupName)`:
	- `DENSE_RANK()`: Это оконная функция, которая присваивает ранг каждой уникальной группе значений. В отличие от `ROW_NUMBER()`, `DENSE_RANK()` присваивает одинаковый ранг одинаковым значениям в группе, и между рангами нет "пропусков". Например, если у вас есть группы 'A', 'A', 'B', 'C', 'C', `DENSE_RANK()` присвоит им ранги 1, 1, 2, 3, 3 соответственно.
	- `OVER (ORDER BY GroupName)`: Эта часть определяет "окно" или набор строк, над которым будет работать оконная функция. `ORDER BY GroupName` гарантирует, что группы будут ранжироваться в алфавитном порядке их имен.
2. `CHAR(96 + DENSE_RANK() OVER (ORDER BY GroupName))`:
	- Функция `DENSE_RANK()` вернет целые числа, начиная с 1 (1, 2, 3…).
	- Мы хотим преобразовать эти числа в буквы (a, b, c…). Код ASCII для 'a' равен 97.
	- Поэтому, если мы возьмем ранг (например, 1 для первой группы) и прибавим к нему 96, мы получим 97. `CHAR(97)` вернет 'a'.
	- Для ранга 2, 96 + 2 = 98, `CHAR(98)` вернет 'b', и так далее.
3. `AS GroupLetter`: Присваивает псевдоним `GroupLetter` новому столбцу, содержащему присвоенную букву группы.
4. `FROM YourTable AS T`: Указывает таблицу, из которой выбираются данные, с псевдонимом T.
5. `ORDER BY GroupLetter, T.GroupName, T.Value`: Это упорядочивает конечный результат. Сначала по присвоенной букве группы, затем по фактическому имени группы (чтобы все записи одной группы были вместе), и, опционально, по другим столбцам для внутреннего порядка в группе.
---
## Пример Использования:
Представьте, что у вас есть такая таблица Products:

| ProductID | ProductName | Category    |
| --------- | ----------- | ----------- |
| 1         | Laptop      | Electronics |
| 2         | Mouse       | Electronics |
| 3         | Keyboard    | Electronics |
| 4         | Book        | Books       |
| 5         | Magazine    | Books       |
| 6         | Shirt       | Apparel     |
Вы хотите сгруппировать по Category и присвоить каждой категории букву.
```sql
SELECT  
    CHAR(96 + DENSE_RANK() OVER (ORDER BY Category)) AS GroupLetter,  
    P.Category,  
    P.ProductName  
FROM  
    Products AS P  
ORDER BY  
    GroupLetter, P.Category, P.ProductName;  
```  

Результат будет выглядеть примерно так:

| GroupLetter | Category    | ProductName |
| ----------- | ----------- | ----------- |
| a           | Apparel     | Shirt       |
| b           | Books       | Book        |
| b           | Books       | Magazine    |
| c           | Electronics | Keyboard    |
| c           | Electronics | Laptop      |
| c           | Electronics | Mouse       |
Таким образом, вы эффективно присваиваете каждой уникальной группе последовательную букву и выводите связанные с ней записи.
# Порядковый Номер Записи Внутри Каждой Группы
Можете вывести порядковый номер записи внутри каждой группы, используя оконную функцию ROW_NUMBER() совместно с PARTITION BY.
Вот как это можно сделать :

```sql
SELECT  
    CHAR(96 + DENSE_RANK() OVER (ORDER BY GroupName)) AS GroupLetter,  
    T.GroupName,  
    ROW_NUMBER() OVER (PARTITION BY GroupName ORDER BY Value) AS RowNumInGroup,  
    T.Value  
FROM  
    YourTable AS T  
ORDER BY  
    GroupLetter, T.GroupName, RowNumInGroup;  
```  
---
Что нового и объяснение:

1. ROW_NUMBER() OVER (PARTITION BY GroupName ORDER BY Value):
	- ROW_NUMBER(): Эта оконная функция присваивает последовательный уникальный номер каждой строке в своем "окне".
	- PARTITION BY GroupName: Это ключевое отличие. PARTITION BY GroupName делит ваш набор данных на отдельные разделы (группы) на основе значений в столбце GroupName. ROW_NUMBER() будет перезапускаться (начинать с 1) для каждой новой группы.
	- ORDER BY Value: Внутри каждой группы (раздела) строки будут упорядочиваться по столбцу Value (или любому другому столбцу, по которому вы хотите определить порядок нумерации внутри группы). Если у вас несколько столбцов, по которым нужно упорядочить, можете указать их через запятую.
	- AS RowNumInGroup: Это псевдоним для нового столбца, который будет содержать порядковый номер записи внутри группы.
---
## Пример Использования С Products Таблицей:
Допустим, у нас та же таблица Products:

| ProductID | ProductName | Category    |
| --------- | ----------- | ----------- |
| 1         | Laptop      | Electronics |
| 2         | Mouse       | Electronics |
| 3         | Keyboard    | Electronics |
| 4         | Book        | Books       |
| 5         | Magazine    | Books       |
| 6         | Shirt       | Apparel     |
Мы хотим пронумеровать продукты внутри каждой категории, упорядочивая их по ProductName.
```sql
SELECT
    CHAR(96 + DENSE_RANK() OVER (ORDER BY Category)) AS GroupLetter,
    P.Category,
    ROW_NUMBER() OVER (PARTITION BY Category ORDER BY ProductName) AS ProductNumInCat,
    P.ProductName
FROM
    Products AS P
ORDER BY
    GroupLetter, P.Category, ProductNumInCat;
```
Результат будет выглядеть так:

| GroupLetter | Category    | ProductNumInCat | ProductName |
| ----------- | ----------- | --------------- | ----------- |
| a           | Apparel     | 1               | Shirt       |
| b           | Books       | 1               | Book        |
| b           | Books       | 2               | Magazine    |
| c           | Electronics | 1               | Keyboard    |
| c           | Electronics | 2               | Laptop      |
| c           | Electronics | 3               | Mouse       |
Как видите, ProductNumInCat начинается с 1 для каждой новой категории.
Эта техника очень полезна для генерации отчетов, нумерации элементов в группах или выполнения других аналитических задач.