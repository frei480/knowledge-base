---
tags:
  - python
Автор: Jakub Beránek
Ссылка: https://www.youtube.com/watch?v=OFRLKWacOoA
Тип: Пересказ видео
Название: Writing Python like it's Rust - more robust code with type hints
---
00:04 Введение

• Лекция о том, как писать на Python, как на Rust, основана на статье автора.
• Автор Секу Беране́к, аспирант в области компьютерных наук, преподаватель и исследователь.
• Опыт работы с Python более десяти лет, но с проблемами в больших программах.

00:57 Проблемы с Python и переход на Rust

• Ошибки и сложности в отладке кода на Python.
• Переход на Rust, который стал любимым языком программирования.
• Желание перенести опыт Rust на Python.

02:36 Использование подсказок типов

• Подсказки типов помогают в понимании кода и документации.
• Использование подсказок типов в интерфейсах функций и классах данных.
• Подсказки типов помогают в автодополнении и навигации в IDE.

04:33 Преимущества подсказок типов

• Подсказки типов улучшают понимание кода и его документирование.
• Помогают в автодополнении и навигации в IDE.
• Помогают понять, когда код становится слишком сложным и требует разделения.

07:22 Проверка типов и инструменты

• Проверка типов с помощью Pyrite или Mypy.
• Настройка проверки типов в непрерывной интеграции.
• Проверка типов помогает быстро находить и исправлять ошибки, что ускоряет процесс разработки.

09:48 Проверка типов и тестирование

• Проверка типов кода занимает всего секунду, в отличие от запуска тестового набора, который занимает 30 секунд.
• Проверка типов позволяет быстрее получать обратную связь.
• Можно использовать библиотеки для проверки типов во время выполнения программы.

10:39 Преимущества подсказок типов

• Подсказки типов повышают уверенность в коде и доверие к коду других людей.
• Они помогают при рефакторинге и изменении кода.
• Подсказки типов не имеют значительных недостатков для автора.

11:37 Недостатки подсказок типов

• Некоторые люди считают, что подсказки типов требуют больше символов для набора.
• Подсказки типов могут быть бесполезны для одноразового кода.
• Важно использовать подсказки типов даже для прототипов, чтобы избежать ошибок в продакшене.

12:25 Ложное чувство безопасности

• Подсказки типов создают ложное чувство безопасности, так как система типов Python не так надежна, как в других языках.
• Несмотря на это, использование подсказок типов остается хорошей идеей.

13:25 Классы данных

• Использование классов данных помогает лучше понимать код и улучшает автодополнение в IDE.
• Классы данных обеспечивают более надежное именование и навигацию в коде.

15:16 Надежность кода

• Надежный код не допускает ошибок и сложно использовать неправильно.
• В Rust ошибки компиляции предотвращают неправильное использование кода.
• В Python ошибки проверки типов помогают, но система типов менее развита.

17:07 Примеры улучшения надежности кода

• Пример с функциями для получения номеров автомобиля и водителя.
• Разделение типов номеров улучшает надежность кода и документацию.
• Пример с клиентом TCP/IP, где инварианты описаны в документации, но могут быть нарушены.

19:44 Проблемы с API и их решение

• Забывание вызвать функцию закрытия может не привести к сбою, но это ошибка.
• Можно изменить API, чтобы такие ситуации были невозможны.
• Использование контекстного менеджера для создания и закрытия клиента.

20:38 Недопустимые состояния и их предотвращение

• Невозможно подключиться дважды с новым API.
• Ошибки можно исправить, сохраняя переменную клиента вне блока ожидания.
• В Rust это невозможно, в Python сложнее, но ошибки можно предотвратить.

21:37 Аутентификация и создание запросов

• Ошибки при вызове метода build без аутентификации.
• Создание отдельных типов запросов с токенами и паролями.
• Изменение конструктора для возврата другого типа при настройке токена или пароля.

22:30 Пример с виртуальной кнопкой

• Создание состояния кнопки с атрибутами, такими как наличие руки и таймер.
• Проблемы с отображением некорректных данных.
• Переписывание кода как конечного автомата для четкого перечисления всех состояний.

24:28 Заключение и советы

• Идея надежности заключается в затруднении ошибок в API.
• Создание "ловушки успеха" для легкого правильного выполнения.
• Использование подсказок типов и классов данных для усложнения неправильного использования кода.

25:50 Вопросы и ответы

• Вопрос о предпочтительном использовании Rust вместо Python.
• Rust используется для низкоуровневых задач и распределенных систем.
• Вопрос о влиянии подсказок типов на производительность Python.
• Подсказки типов не используются для повышения производительности в CPython.

