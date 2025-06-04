---
tags:
  - python
---

Вот улучшенная версия скрипта с использованием генератора для чтения файла и обработки данных порциями:

```python
import os
from concurrent.futures import ThreadPoolExecutor
import csv
from itertools import islice

def file_generator(input_file, batch_size=1000):
    """Генератор для чтения файла порциями"""
    with open(input_file, 'r', encoding='utf-8') as f:
        while True:
            batch = list(islice(f, batch_size))
            if not batch:
                break
            for line in batch:
                cleaned_line = line.strip()
                if cleaned_line:
                    yield cleaned_line

def check_file_existence(args):
    """Проверяет существование файла"""
    base_path, file_path = args
    full_path = os.path.join(base_path, file_path) if not os.path.isabs(file_path) else file_path
    
    try:
        return {
            'FilePath': full_path,
            'Exists': os.path.exists(full_path),
            'Error': None
        }
    except Exception as e:
        return {
            'FilePath': full_path,
            'Exists': False,
            'Error': str(e)
        }

def process_batch(batch, base_path):
    """Обрабатывает порцию файлов"""
    with ThreadPoolExecutor() as executor:
        return list(executor.map(
            check_file_existence,
            [(base_path, file_path) for file_path in batch]
        ))

def main():
    # Конфигурация
    input_file = 'file_list.txt'
    output_file = 'results.csv'
    network_path = r'X:\\'
    batch_size = 5000  # Размер порции для обработки
    
    # Инициализация генератора
    files_gen = file_generator(input_file)
    
    # Обработка файлов порциями
    with open(output_file, 'w', newline='', encoding='utf-8') as csvfile:
        writer = csv.DictWriter(csvfile, fieldnames=['FilePath', 'Exists', 'Error'])
        writer.writeheader()
        
        while True:
            batch = list(islice(files_gen, batch_size))
            if not batch:
                break
            
            # Параллельная обработка порции
            results = process_batch(batch, network_path)
            
            # Запись результатов
            writer.writerows(results)
            print(f'Обработано {len(batch)} файлов...')
    
    print('Проверка завершена!')

if __name__ == '__main__':
    main()
```

Основные улучшения:

1. **Генератор файлов** (`file_generator`):
- Читает файл порциями заданного размера
- Очищает и фильтрует строки
- Не загружает весь файл в память

2. **Пакетная обработка**:
- Обработка данных порциями (по 5000 файлов за раз)
- Запись результатов после обработки каждой порции
- Оптимальное использование памяти

3. **Гибкая настройка**:
- Размер порции можно регулировать через `batch_size`
- Отдельная функция для обработки пакета (`process_batch`)
- Возможность добавления прогресс-бара

4. **Эффективное использование ресурсов**:
- Потоки создаются для каждой порции отдельно
- Автоматическое освобождение ресурсов после обработки
- Минимизация потребления памяти

Для добавления прогресс-бара можно использовать `tqdm`:

```python
from tqdm import tqdm

# Модифицируем main()
def main():
    ...
    with open(output_file, 'w', ...) as csvfile:
        ...
        with tqdm(desc="Обработка файлов", unit=" файлов") as pbar:
            while True:
                batch = list(islice(files_gen, batch_size))
                ...
                pbar.update(len(batch))
                ...
```

Это решение особенно эффективно для:
- Очень больших файлов (миллионы записей)
- Систем с ограниченной оперативной памятью
- Долгих сетевых проверок, где важна пакетная обработка