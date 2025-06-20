---
tags:
  - SQL
Тип: Статья
Автор: Craig Freedman
Ссылка: https://learn.microsoft.com/en-us/archive/blogs/craigfr/ranking-functions-row_number
---
Четыре функции ранжирования: `ROW_NUMBER`, `RANK`, `DENSE_RANK` и `NTILE` появились в SQL Server 2005 и отличаются от обычных скалярных функций тем, что результат, который они выдают для строки, зависит от других строк выборки. От агрегатных функций они отличаются тем, что возвращают только одну строку для каждой строки на входе, т. е. они не объединяют набор строк в одну. В этой статье мы рассмотрим `ROW_NUMBER` - самую простую из всех функций ранжирования.
Для примеров будем использовать следующий набор данных:
```sql
CREATE TABLE T (PK INT IDENTITY, A INT, B INT, C INT)
CREATE UNIQUE CLUSTERED INDEX TPK ON T(PK)
INSERT T VALUES (0, 1, 8)
INSERT T VALUES (0, 3, 6)
INSERT T VALUES (0, 5, 4)
INSERT T VALUES (0, 7, 2)
INSERT T VALUES (0, 9, 0)
INSERT T VALUES (1, 0, 9)
INSERT T VALUES (1, 2, 7)
INSERT T VALUES (1, 4, 5)
INSERT T VALUES (1, 6, 3)
INSERT T VALUES (1, 8, 1)
```
Начнем со следующего простого примера:
```sql
SELECT *, ROW_NUMBER() OVER (ORDER BY B) AS RowNumber FROM T
```
Этот запрос упорядочивает строки таблицы по столбцу B и присваивает этим строкам последовательные номера (так же, как столбец с identity). Результат выглядит так:
```
PK    A     B     C     RowNumber
----- ----- ----- ----- ----------
6     1     0     9     1
1     0     1     8     2
7     1     2     7     3
2     0     3     6     4
8     1     4     5     5
3     0     5     4     6
9     1     6     3     7
4     0     7     2     8
10    1     8     1     9
5     0     9     0     10
```

У этого запроса будет вот такой простой план:

```
  |--Sequence Project(DEFINE:([Expr1003]=row_number))
       |--Compute Scalar(DEFINE:([Expr1005]=(1)))
            |--Segment [GROUP BY:()]
                 |--Sort(ORDER BY:([T].[B] ASC))
                      |--Clustered Index Scan(OBJECT:([T].[TPK]))
```

Этот план просматривает таблицу, сортирует ее по столбцу B, а затем оператор Sequence Project присваивает каждой строке последовательные номера.

Оператор Segment обычно разбивает соседние строки на группы связанных строк. В нашем примере в этом нет необходимости, поскольку все строки находятся в одной группе. Обратите внимание, что в текстовом варианте представления плана для оператора Segment столбцы GROUP BY не отображаются; эта информация доступна только для SHOWPLAN_ALL, SHOWPLAN_XML или в графическом представлении плана. Я добавил аннотацию в текстовый план, чтобы явно показать, что у оператора Segment нет столбцов GROUP BY. Сейчас мы рассмотрим пример, в котором без оператора Segment не обойтись.

Оператор Compute Scalar также тут не нужен и был удален из подобных планов начиная с SQL Server 2008.

ROW_NUMBER (и другие функции ранжирования) также поддерживают разделение строк на группы с последующим вычислением функции отдельно для каждой группы:

```sql
SELECT *, ROW_NUMBER() OVER (PARTITION BY A ORDER BY B) AS RowNumber FROM T
```

Поскольку столбец A имеет два значения (0 и 1), этот запрос разбивает строки таблицы на две группы и присваивает номера строк отдельно каждой группе.

```
PK    A     B     C     RowNumber
----- ----- ----- ----- ----------
1     0     1     8     1
2     0     3     6     2
3     0     5     4     3
4     0     7     2     4
5     0     9     0     5
6     1     0     9     1
7     1     2     7     2
8     1     4     5     3
9     1     6     3     4
10    1     8     1     5
```

План этого запроса практически идентичен предыдущему плану:

```
  |--Sequence Project(DEFINE:([Expr1003]=row_number))
       |--Compute Scalar(DEFINE:([Expr1005]=(1)))
            |--Segment [GROUP BY:([T].[A])]
                 |--Sort(ORDER BY:( [T].[A] ASC, [T].[B] ASC))
                      |--Clustered Index Scan(OBJECT:([T].[TPK]))
```

Обратите внимание, что в этом плане сортировка осуществляется по столбцам A и B, а затем оператор Segment группирует строки по столбцу A. Sequence Project присваивает каждой строке последовательные номера, как и в первом примере, но сбрасывает счетчик строк в начале каждой группы. Заметно некоторое сходство между тем, как работает группировка в этом плане, и тем, как группировка работает в плане с оператором [Агрегат потока (Stream Aggregate)](https://mssqlforever.blogspot.com/2024/02/stream-aggregate.html).

Можно объединить в одном запросе несколько функций ROW_NUMBER (или несколько любых других функций ранжирования) с разными предложениями ORDER BY:
```sql
SELECT *,
	ROW_NUMBER() OVER (ORDER BY B) AS RowNumber_B,
	ROW_NUMBER() OVER (ORDER BY C) AS RowNumber_C
FROM T
```

```
PK    A     B     C     RowNumber_B  RowNumber_C
----- ----- ----- ----- ------------ ------------
5     0     9     0     10           1
10    1     8     1     9            2
4     0     7     2     8            3
9     1     6     3     7            4
3     0     5     4     6            5
8     1     4     5     5            6
2     0     3     6     4            7
7     1     2     7     3            8
1     0     1     8     2            9
6     1     0     9     1            10
```
План этого запроса аналогичен плану первого запроса, только он повторяется один раз для каждой функции ранжирования с разным порядком сортировки для каждого предложения ORDER BY:
```
  |--Sequence Project(DEFINE:([Expr1004]=row_number))
       |--Compute Scalar(DEFINE:([Expr1008]=(1)))
            |--Segment [GROUP BY:()]
                 |--Sort(ORDER BY:( [T].[C] ASC))
                      |--Sequence Project(DEFINE:([Expr1003]=row_number))
                           |--Compute Scalar(DEFINE:([Expr1006]=(1)))
                                |--Segment [GROUP BY:()]
                                     |--Sort(ORDER BY:( [T].[B] ASC))
                                          |--Clustered Index Scan(OBJECT:([T].[TPK]))
```
Обратите внимание, что предложение ORDER BY связанно со своей функцией ранжирования и определяет порядок только для неё. Такие ORDER BY не определяют порядок, в котором строки будут возвращены запросом. Чтобы гарантировать необходимый порядок строк запроса нужно явно добавить предложение ORDER BY в запрос:
```sql
SELECT *, ROW_NUMBER() OVER (ORDER BY B) AS RowNumber FROM T ORDER BY B
```
В этом запросе дополнительное предложение ORDER BY не повлияет на план, поскольку оптимизатор сообразит, что данные уже отсортированы по столбцу B после вычисления функции ROW_NUMBER.

К сожалению, если у нас есть несколько функций ROW_NUMBER со своими ORDER BY и есть ORDER BY у всего запроса, оптимизатор не станет разбираться будет ли оптимальной вычислять функции ROW_NUMBER в другом порядке. Например, следующий запрос выполнит дополнительную сортировку:

```sql
SELECT *,
	ROW_NUMBER() OVER (ORDER BY B) AS RowNumber_B,
	ROW_NUMBER() OVER (ORDER BY C) AS RowNumber_C
FROM T
ORDER BY B
```

```
  |--Sort(ORDER BY:([T].[B] ASC))
       |--Sequence Project(DEFINE:([Expr1004]=row_number))
            |--Compute Scalar(DEFINE:([Expr1008]=(1)))
                 |--Segment
                      |--Sort(ORDER BY:([T].[C] ASC))
                           |--Sequence Project(DEFINE:([Expr1003]=row_number))
                                |--Compute Scalar(DEFINE:([Expr1006]=(1)))
                                     |--Segment
                                          |--Sort(ORDER BY:([T].[B] ASC))
                                               |--Clustered Index
```

Но, если мы просто поменяем в выборке порядок двух функций ROW_NUMBER, дополнительная сортировка исчезнет:

```sql
SELECT *,
ROW_NUMBER() OVER (ORDER BY C) AS RowNumber_C,
ROW_NUMBER() OVER (ORDER BY B) AS RowNumber_B
FROM T
ORDER BY B
```

```
  |--Sequence Project(DEFINE:([Expr1004]=row_number))
       |--Compute Scalar(DEFINE:([Expr1008]=(1)))
            |--Segment
                 |--Sort(ORDER BY:([T].[B] ASC))
                      |--Sequence Project(DEFINE:([Expr1003]=row_number))
                           |--Compute Scalar(DEFINE:([Expr1006]=(1)))
                                |--Segment
                                     |--Sort(ORDER BY:([T].[C] ASC))
                                          |--Clustered Index Scan(OBJECT:([T].[TPK]))
```

Как и в случае с агрегатом потока, SQL Server может исключить сортировку для оператора Sequence Project, если существует индекс:

```sql
CREATE INDEX TB ON T(B)
SELECT PK,
	ROW_NUMBER() OVER (ORDER BY B) AS RowNumber_B
FROM T
```

```
  |--Sequence Project(DEFINE:([Expr1003]=row_number))
       |--Compute Scalar(DEFINE:([Expr1005]=(1)))
            |--Segment
                 |--Index Scan(OBJECT:([T].[TB]), ORDERED FORWARD)
```

Однако, поскольку план запроса с несколькими ROW_NUMBER и разными ORDER BY наслаивает несколько операторов Sequence Project поверх просмотра, SQL Server может использовать в таком плане только один индекс. Получается, в плане такого запроса должна быть сортировка. Даже если план использует индекс вместо одной из сортировок, если он не включает все столбцы запроса, оптимизатор будет выбирать использовать ли индекс вместе с Bookmark Lookup или использовать кластерный индекс и выполнить дополнительную сортировку.

Нужно отметить, что SQL Server не использует функции ROW_NUMBER без предложения ORDER BY. Как правило, использование функции ранжирования без предложения ORDER BY бессмысленно, поскольку результаты не детерминированы. Однако в некоторых случаях можно обойтись и без предложения ORDER BY. Например, можно использовать ROW_NUMBER для присвоения значений идентификаторов. В статье [Row numbers with nondeterministic order](https://sqlperformance.com/2019/11/t-sql-queries/row-numbers-with-nondeterministic-order) Itzik Ben‑Gan обсуждает подобную проблематику и варианты решений.
В следующем посте я рассмотрю другие функции ранжирования, а именно RANK, DENSE_RANK и NTILE.