---
tags:
  - SQL
Тип: Статья
Автор: Craig Freedman
Ссылка: https://learn.microsoft.com/en-us/archive/blogs/craigfr/ranking-functions-rank-dense_rank-and-ntile
---
В [[SQL Функции ранжирования - ROW_NUMBER|предыдущей статье]]  обсуждалась функция [ROW_NUMBER](https://learn.microsoft.com/ru-ru/sql/t-sql/functions/row-number-transact-sql?view=sql-server-ver16). Сейчас же мы рассмотрим другие функции ранжирования: [RANK](https://learn.microsoft.com/ru-ru/sql/t-sql/functions/rank-transact-sql?view=sql-server-ver16), [DENSE_RANK](https://learn.microsoft.com/ru-ru/sql/t-sql/functions/dense-rank-transact-sql?view=sql-server-ver16) и [NTILE](https://learn.microsoft.com/ru-ru/sql/t-sql/functions/ntile-transact-sql?view=sql-server-ver16). Начнем с RANK и DENSE_RANK. Эти функции по функциональности и реализации аналогичны ROW_NUMBER. Разница заключается в том, что ROW_NUMBER присваивает уникальные возрастающие значения каждой строке, не обращая внимания на повторение значений выражения сортировки, тогда как RANK и DENSE_RANK присваивают одинаковые значения строкам с одинаковым значением выражения сортировки. Разница между функциями RANK и DENSE_RANK заключается в том, как значения присваиваются строкам. Проще всего проиллюстрировать разницу между всеми этими функциями на простом примере:

```sql
CREATE TABLE T (PK INT IDENTITY, A INT, B INT, C INT)
CREATE UNIQUE CLUSTERED INDEX TPK ON T(PK)
INSERT T VALUES (0, 1, 6)
INSERT T VALUES (0, 1, 4)
INSERT T VALUES (0, 3, 2)
INSERT T VALUES (0, 3, 0)
INSERT T VALUES (1, 0, 7)
INSERT T VALUES (1, 0, 5)
INSERT T VALUES (0, 2, 3)
INSERT T VALUES (0, 2, 1)
SELECT *,
	ROW_NUMBER() OVER (ORDER BY B) AS RowNumber,
	RANK() OVER (ORDER BY B) AS Rank,
	DENSE_RANK() OVER (ORDER BY B) AS DenseRank
FROM T
```

```
PK    A     B     C     RowNumber  Rank       DenseRank
----- ----- ----- ----- ---------- ---------- ----------
5     1     0     7     1          1          1
6     1     0     5     2          1          1
1     0     1     6     3          3          2
2     0     1     4     4          3          2
7     0     2     3     5          5          3
8     0     2     1     6          5          3
3     0     3     2     7          7          4
4     0     3     0     8          7          4
```

Обратите внимание, функция ROW_NUMBER игнорирует повторяющиеся значения столбца B и присваивает уникальные значения от 1 до 8 всем строкам. Функции RANK и DENSE_RANK присваивают одно и то же значение каждому дублю строк. Причём, функция RANK подсчитывает повторяющиеся строки, даже если она присваивает одно и то же значение повторяющейся строке, тогда как функция DENSE_RANK не учитывает повторяющиеся строки. Например, функции RANK и DENSE_RANK присваивают ранг 1 первым двум строкам, но функция RANK присваивает ранг 3 третьей строке (так как это третья по счёту строка), а функция DENSE_RANK присваивает ранг 2 третьей строке (так как она содержит второе уникальное значение для столбца B). Как видно, максимальное значение, возвращаемое функцией DENSE_RANK, равно количеству уникальных значений в столбце B.

План запроса для всех этих функций один, и поскольку все они используют одни и те же предложения PARTITION BY и ORDER BY, один оператор Sequence Project может вычислить все функции одновременно:

```
**       |--Segment [GROUP BY:([T].[B])]
**            |--Segment [GROUP BY:()]
                 |--Sort(ORDER BY:([T].[B] ASC))
                      |--Clustered Index Scan(OBJECT:([T].[TPK]))
```

Единственная существенная разница между этим планом и планом с одной ROW_NUMBER — это дополнительный оператор Segment с группировкой по столбцу B. Его задача обнаружить связи и принудить оператор Sequence Project, выводить одно и то же значение при генерации новых значений функциями RANK и DENSE_RANK.

Все [[SQL Функции ранжирования - ROW_NUMBER|рецепты из предыдущей статьи о ROW_NUMBER]] (использование индекса во избежание сортировки или влияние смешивания функций ранжирования с различными предложениями PARTITION BY и/или ORDER BY) также применимы к функциям RANK и DENSE_RANK, поэтому их повторять не будем.

Теперь давайте посмотрим на функцию NTILE. Эта функция разбивает набор данных на N групп одинакового размера. Чтобы определить, сколько строк принадлежит каждой группе, SQL Server должен сначала определить общее количество строк в наборе. Если функция NTILE включает предложение PARTITION BY, SQL Server должен вычислить количество строк в каждой группе отдельно. Зная количество строк в каждой группе, мы можем записать функцию NTILE как:

```sql
NTILE(N) := (N * (ROW_NUMBER() - 1) / COUNT(*)) + 1
```

где `COUNT(*)` — количество строк в каждой группе. В этом уравнении используются -1 и +1, поскольку функции ROW_NUMBER и NTILE отсчитываются от 1, а не от 0.

Мы уже знаем, как вычислить функцию ROW_NUMBER, поэтому интересно, как вычислить функцию `COUNT(*)`. Оказывается, SQL Server использует предложение OVER, которое мы уже видели в функциях ранжирования, и почти во всех агрегатных функциях. Этот тип агрегатной функции иногда называют «оконной агрегатной функцией», при использовании которой вместо одной строки для каждой группы возвращается один и тот же агрегированный результат для каждой строки группы. Давайте рассмотрим это на примере:

```sql
SELECT *, COUNT(*) OVER (PARTITION BY A) AS Cnt FROM T
```

```
PK    A     B     C     Cnt
----- ----- ----- ----- ----------
1     0     1     6     6
2     0     1     4     6
3     0     3     2     6
4     0     3     0     6
7     0     2     3     6
8     0     2     1     6
5     1     0     7     2
6     1     0     5     2
```

В отличие от скалярного агрегированного запроса, который может возвращать только агрегированные значения функции или агрегированного запроса с GROUP BY, который может возвращать только столбцы из GROUP BY и агрегированные значения, этот запрос возвращает все столбцы таблицы. Более того, поскольку каждая строка в группе получает один и тот же результат, предложение ORDER BY в этом сценарии не поддерживается. Теперь давайте посмотрим на план этого запроса:

```
  |--Nested Loops(Inner Join)
       |--Table Spool
       |    |--Segment [GROUP BY:([T].[A])]
       |         |--Sort(ORDER BY:([T].[A] ASC))
       |              |--Clustered Index Scan(OBJECT:([T].[TPK]))
       |--Nested Loops(Inner Join, WHERE:((1)))
            |--Compute Scalar(DEFINE:([Expr1003]=CONVERT_IMPLICIT(int,[Expr1005],0)))
            |    |--Stream Aggregate(DEFINE:([Expr1005]=Count(*)))
            |         |--Table Spool
            |--Table Spool
```

Хотя этот план сложнее, чем обычный план с агрегатами, на самом деле он тоже простой. Операторы Sort и Segment разбивают строки на группы также, как это делает агрегат потока. Table Spool над оператором Segment представляет собой особый тип буфера подвыражения, который принято называть Segment Spool. Он копирует все строки одной группы, а затем, когда оператор Segment начинает новую группу, он отправляет строку в самый верхний [Nested Loops](https://mssqlforever.blogspot.com/2024/04/nested-loops-join.html), который выполняет свою итерацию. На этом этапе Table Spool выдаёт строки из текущей группы, а Stream Aggregate их подсчитывает. Наконец, Stream Aggregate возвращает счетчик нижнему Nested Loops, который соединяет этот счетчик со строками из текущей группы - Table Spool выдаёт эти строки во второй раз.

Обратите внимание, что для этого плана требуется Segment Spool, поскольку невозможно заранее узнать, сколько строк будет в одной группе с текущей строкой, без просмотра по всей группе, а после этого, посчитав строки, нужно каким-то образом вернуться и применить этот счетчик к каждой из исходных строк. Spool тут дает возможность вернуться назад. Типичный агрегатный запрос с предложением GROUP BY не имеет таких проблем, поскольку после вычисления агрегатной функции (будь то COUNT или любая другая функция) он просто выводит результат, не возвращаясь к исходным строкам.

Теперь посмотрим простенький пример с вычислением функции NTILE. Запрос ниже группирует строки по столбцу A, а затем для каждой группы вычисляет, какие строки находятся ниже медианы (присваивается 1), а какие строки выше медианы (присваивается 2):

```sql
SELECT *, NTILE(2) OVER (PARTITION BY A ORDER BY B) AS NTile FROM T
```

Вот результат запроса:

```
PK    A     B     C     NTile
----- ----- ----- ----- ------
1     0     1     6     1
2     0     1     4     1
7     0     2     3     1
8     0     2     1     2
3     0     3     2     2
4     0     3     0     2
5     1     0     7     1
6     1     0     5     2
```

А вот план этого запроса:
```
**  |--Sequence Project(DEFINE:([Expr1003]=ntile))
       |--Compute Scalar(DEFINE:([Expr1007]=(1)))
            |--Segment [GROUP BY:([T].[A])]
**                 |--Nested Loops(Inner Join)
                      |--Table Spool
                      |    |--Segment [GROUP BY:([T].[A])]
                      |         |--Sort(ORDER BY:([T].[A] ASC, [T].[B] ASC))
                      |              |--Clustered Index Scan(OBJECT:([T].[TPK]))
                      |--Nested Loops(Inner Join, WHERE:((1)))
                           |--Stream Aggregate(DEFINE:([Expr1004]=Count(*)))
                           |    |--Table Spool
                           |--Table Spool
```

Этот план в основном такой же, как предыдущий с `COUNT(*)`. Единственное отличие в добавлении дополнительных операторов Sequence Project и Sequence, которые вычисляют функцию NTILE путем вычисления ROW_NUMBER и применения формулы, показанной выше. Используя эту формулу, мы могли бы вычислить NTILE вручную следующим образом:
```sql
SELECT *, (2*(RowNumber-1)/Cnt)+1 AS MyNTile
FROM
	(
	    SELECT *,
			ROW_NUMBER() OVER (PARTITION BY A ORDER BY B) AS RowNumber,
			COUNT(*) OVER (PARTITION BY A ) AS Cnt,
			NTILE(2) OVER (PARTITION BY A ORDER BY B) AS NTile
		FROM T
		) TSeq
```

К сожалению, оптимизатор не знает, что он может использовать один и тот же фрагмент плана для вычисления функций ранжирования и `COUNT(*)` оконной агрегатной функции, поэтому в итоге выбирает неэффективный план, который вычисляет `COUNT(*)` дважды:

```
  |--Compute Scalar(DEFINE:([Expr1006]=((2)*([Expr1003]-(1)))/CONVERT_IMPLICIT(bigint,[Expr1004],0)+(1)))
       |--Nested Loops(Inner Join)
            |--Table Spool
            |    |--Segment [GROUP BY:([T].[A])]
            |         |--Sequence Project(DEFINE:([Expr1003]=row_number, [Expr1005]=ntile))
            |              |--Compute Scalar(DEFINE:([Expr1010]=(1)))
            |                   |--Segment [GROUP BY:([T].[A])]
            |                        |--Nested Loops(Inner Join)
            |                             |--Table Spool
            |                             |    |--Segment [GROUP BY:([T].[A])]
            |                             |         |--Sort(ORDER BY:([T].[A] ASC, [T].[B] ASC))
            |                             |              |--Clustered Index Scan(OBJECT:([T].[TPK]))
            |                             |--Nested Loops(Inner Join, WHERE:((1)))
            |                                  |--Stream Aggregate(DEFINE:([Expr1007]=Count(*)))
            |                                  |    |--Table Spool
            |                                  |--Table Spool
            |--Nested Loops(Inner Join, WHERE:((1)))
                 |--Compute Scalar(DEFINE:([Expr1004]=CONVERT_IMPLICIT(int,[Expr1012],0)))
                 |    |--Stream Aggregate(DEFINE:([Expr1012]=Count(*)))
                 |         |--Table Spool
                 |--Table Spool
```

Я оставлю анализ этого плана вам в качестве упражнения. Наслаждайтесь!