# 1. Merge Sort
> [!NOTE]-
> Идея:
> > 1. Делим массив на 2 части и сортируем, далее рекурсивно разбиваем каждую часть пополам и сортируем.
> > 2. Сливаем отсортированные части вместе.
>
> ```python
> def merge(a:list, b:list, to:list):
> 	i=0
> 	j=0
> 	while(i < len(a) or j < len(b)):
> 		if (j == len(b)-1 or (i < n and a[i] < b[j])):
> 			to[i+j] = a[i]
> 			i+=1
> 		else:
> 			to[i+j] = b[j]
> 			j+=1
> 
> def merge_sort(a: list[int]) -> None:
>     if len(a) <= 1:
>         return
>     mid = len(a) // 2
>     left: list[int] = a[:mid]
>     right: list[int] = a[mid:]
>     merge_sort(left)
>     merge_sort(right)
>     merge(left, right, a)
> ```

# 2. Quick Sort (Сортировка Хоара)
> [!NOTE]-
> Идея:
> > 1. В канонической реализации выбираем x - pivot случайно.
> > 2. Разбиваем массив на три части <x, =x, >x
> > 3. Рекурсивно сортируем каждую из частей.
>
> ```python
> def qsort(a: list[int]) -> list[int]:
>     if len(a) <= 1:
>         return a
>     pivot = a[len(a) // 2]
>     left: list[int] = [x for x in a if x < pivot]
>     mid: list[int] = [x for x in a if x == pivot]
>     right: list[int] = [x for x in a if x > pivot]
>     return qsort(left) + mid + qsort(right)
> ```

# 3. Quick Select
> [!NOTE]-
> Пусть есть массив. Его k-ая порядковая статистика, это тот элемент который стоял бы на k-ом месте после сортировки.
> Задача: найти k-ую порядковую статистику.
> ```python
> def quick_select(a, k):
> 	if len(a) == 1:
> 	        return a[0]
> 	pivot = a[len(a) // 2]
>     left: list[int] = [x for x in a if x < pivot]
>     mid: list[int] = [x for x in a if x == pivot]    
> 	right: list[int] = [x for x in a if x > pivot]
> 	if k <= len(left):
> 		 return quick_select(left, k):
> 	if k <= len(left)+len(mid):
> 		retrun 
> 	return quick_select(right, k-len(left)-len(mid))
> ```
4

# 4. Сортировки Чисел.
## - Сортировка Подсчётом
	Задача: пусть есть массив чисел от 0 до k.
	Решение:
		1. создадим массив `count[k+1] = {0}` заполненный нулями
		2. подсчитаем кол-во вхождений чисел в массиве
		```
		for i=1..n
			++count[a[i]]
		```
		3. вывод:
		```
		for x=0..k
			for i=1..cont[x]
				print(x)
		```
	Асимптотика: работает за $O(n+k)$
## - Стабильная Сортировка Подсчётом
	```
	cnt[k+1]={0}
	for i=1..n
		++cnt[a[i]]
	for x=1..k
		cnt[x]+=cnt[x-1]
	```
	теперь в `cnt[x]` хранится кол-во элементов ≤ x в массиве
	Пример: массив: `{0, 1, 0, 0, 1, 2, 0}`
	```		
	cnt[0] = 4
	cnt[1] = 6
	cnt[2] = 7		
	```
	после сортировки `{0, 0, 0, 0, 1, 1, 2}`
	`cnt[x]` указывает на позицию последнего элемента `x` в отсортированном массиве. последний 0 на 4-ом месте, последняя единица на 6-ом,и т.д.
	проход по массиву:
	```
	for i=n..1
		x=a[i]
		ans[cnt[x]]=x
		--cnt[x]
	```
	Асимптотика: работает за $O(n+k)$

### Задача
Дан массив пар чисел $(a_1,b_1), (a_2,b_2),…(a_n,b_n)$. Необходимо его отсортировать.
условие сортировки:
$$(a_1,b_1) < (a_2,b_2) \iff \begin{cases} x_1 < x_2 \\ x_1 = x_2, y_1<y_2 \end{cases}  $$
Решение:
	1. Отсортируем массив по второму компоненту ($b$)
	2. Результат стабильно отсортируем по $a$

## - Least Significant Digit Sort
сортировка по наименее значимым цифрам.
Задача: отсортировать массив чисел, $a_{i} \in \{0,1,\dots k\}$ k очень большое.
Асимптотика: работает за $O(\log k(n+10))$
Если использовать систему счисления $2^b$ ,тогда последняя цифра числа x - это $x\% 2^b = x\&((1<<b)-1)$
