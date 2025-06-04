---
tags:
  - python
---

Вот решение на Python с использованием многопоточности для проверки файлов на сетевом диске:

```python
import os
from concurrent.futures import ThreadPoolExecutor
import csv

def check_file_existence(args):
    """
    Проверяет существование файла и обрабатывает ошибки
    """
    base_path, file_path = args
    full_path = os.path.join(base_path, file_path) if not os.path.isabs(file_path) else file_path
    
    result = {
        'FilePath': full_path,
        'Exists': False,
        'Error': None
    }
    
    try:
        result['Exists'] = os.path.exists(full_path)
    except Exception as e:
        result['Error'] = str(e)
    
    return result

def main():
    # Конфигурация
    input_file = 'file_list.txt'    # Файл со списком путей
    output_file = 'results.csv'    # Файл для результатов
    network_path = r'X:\\'         # Базовый сетевой путь
    max_workers = 20               # Количество потоков

    # Чтение списка файлов
    with open(input_file, 'r', encoding='utf-8') as f:
        file_list = [line.strip() for line in f if line.strip()]

    # Создаем задачи для пула потоков
    tasks = [(network_path, file_path) for file_path in file_list]

    # Запускаем проверку в многопоточном режиме
    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        results = list(executor.map(check_file_existence, tasks))

    # Сохраняем результаты
    with open(output_file, 'w', newline='', encoding='utf-8') as csvfile:
        fieldnames = ['FilePath', 'Exists', 'Error']
        writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
        
        writer.writeheader()
        for result in results:
            writer.writerow(result)

    print(f'Проверено {len(results)} файлов. Результаты сохранены в {output_file}')

if __name__ == '__main__':
    main()
```

### Ключевые особенности:
1. **Многопоточная обработка** через `ThreadPoolExecutor`
2. **Автоматическая обработка путей** (абсолютных и относительных)
3. **Перехват исключений** при доступе к файлам
4. **Экспорт результатов в CSV** с статусом проверки
5. **Конфигурируемые параметры** (количество потоков, пути)

### Оптимизации для больших объемов данных:
1. **Поточная обработка** файлов
2. **Балансировка нагрузки** между потоками
3. **Лимит одновременных операций** через `max_workers`

### Как использовать:
1. Создайте файл `file_list.txt` с путями (пример):
   ```
   folder1/file1.txt
   /full/path/to/document.docx
   another_folder/image.jpg
   ```

2. Запустите скрипт:
   ```bash
   python file_checker.py
   ```

3. Результаты будут сохранены в `results.csv`:
   ```csv
   FilePath,Exists,Error
   X:\folder1\file1.txt,True,
   \\server\share\full\path\to\document.docx,False,
   X:\another_folder\image.jpg,False,[WinError 59] Сетевая ошибка
   ```

### Советы для улучшения:
- Для UNC-путей добавьте проверку доступности ресурса:
  ```python
  if not os.path.exists(network_path):
      raise Exception("Сетевой путь недоступен")
  ```
- Для очень больших файлов используйте обработку порциями:
  ```python
  from itertools import islice
  
  def batch(iterable, n=1000):
      it = iter(iterable)
      while chunk := list(islice(it, n)):
          yield chunk
  ```
- Добавьте прогресс-бар с помощью `tqdm`:
  ```python
  from tqdm import tqdm
  
  # Замените
  results = list(executor.map(...))
  
  # На
  results = list(tqdm(executor.map(...), total=len(file_list)))
  ```