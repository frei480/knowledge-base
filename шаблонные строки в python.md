---
tags:
  - python
---

Шаблонные строки обеспечивают более простую замену строк, как описано в [**PEP 292.**](https://peps.python.org/pep-0292/) Основной вариант использования строк шаблонов — интернационализация (i18n), поскольку в этом контексте более простой синтаксис и функциональность облегчают перевод, чем другие встроенные средства форматирования строк в Python. В качестве примера библиотеки, построенной на строках шаблонов для i18n, см. [пакет flufl.i18n](https://flufli18n.readthedocs.io/en/latest/) .

Строки шаблонов поддерживают `$`-подстановку.
Модуль [`string`](https://docs.python.org/3/library/string.html#module-string "строка: Обычные строковые операции.")предоставляет [`Template`](https://docs.python.org/3/library/string.html#string.Template "строка.Шаблон")класс
пример:
```python
from string import Template
s = Template('$who likes $what')
s.substitute(who='tim', what='kung pao')
```
```shell
'tim likes kung pao'
```
```python
d = dict(who='tim')
Template('Give $who $100').substitute(d)
```
```console
Traceback (most recent call last):
...
ValueError: Invalid placeholder in string: line 1, col 11
```
```python
Template('$who likes $what').substitute(d)
```
```shell
Traceback (most recent call last):
...
KeyError: 'what'
```
```python
Template('$who likes $what').safe_substitute(d)
'tim likes $what'
```
