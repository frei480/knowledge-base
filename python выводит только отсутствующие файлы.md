---
tags:
  - python
---

Вот модифицированная версия скрипта, которая выводит только отсутствующие файлы:

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
        exists = os.path.exists(full_path)
        return {'FilePath': full_path, 'Exists': exists, 'Error': None}
    except Exception as e:
        return {'FilePath': full_path, 'Exists': False, 'Error': str(e)}

def process_batch(batch, base_path):
    """Обрабатывает порцию файлов"""
    with ThreadPoolExecutor() as executor:
        results = list(executor.map(
            check_file_existence,
            [(base_path, file_path) for file_path in batch]
        ))
    
    # Фильтруем только отсутствующие файлы
    return [result for result in results if not result['Exists']]

def main():
    # Конфигурация
    input_file = 'file_list.txt'
    output_file = 'missing_files.csv'
    network_path = r'X:\\'
    batch_size = 5000
    
    total_missing = 0
    files_processed = 0
    
    # Инициализация генератора
    files_gen = file_generator(input_file)
    
    with open(output_file, 'w', newline='', encoding='utf-8') as csvfile:
        writer = csv.DictWriter(csvfile, fieldnames=['FilePath', 'Error'])
        writer.writeheader()
        
        while True:
            batch = list(islice(files_gen, batch_size))
            if not batch:
                break
            
            missing_files = process_batch(batch, network_path)
            
            # Запись результатов
            for result in missing_files:
                writer.writerow({
                    'FilePath': result['FilePath'],
                    'Error': result['Error'] or ''
                })
            
            files_processed += len(batch)
            total_missing += len(missing_files)
            print(f'Проверено: {files_processed} | Найдено отсутствующих: {total_missing}')

    print(f'\nИтог: {total_missing} файлов отсутствует')
    print(f'Результаты сохранены в: {output_file}')

if __name__ == '__main__':
    main()
```

Ключевые изменения:
1. Фильтрация на уровне обработки каждой порции
2. Упрощенный вывод только необходимых полей (FilePath и Error)
3. Добавлен счетчик отсутствующих файлов
4. Изменены имена выходного файла и столбцов
5. Улучшен вывод статистики в консоль

Пример содержимого выходного файла:
```csv
FilePath,Error
X:\folder1\missing_file.txt,
\\server\share\locked_file.docx,[WinError 5] Доступ запрещен
X:\temp\deleted_file.jpg,
```

Для дополнительной оптимизации можно:
1. Добавить проверку доступности сетевого ресурса перед запуском
2. Использовать прогресс-бар с помощью библиотеки tqdm
3. Реализовать повторные попытки проверки для файлов с ошибками
4. Добавить фильтрацию по типу ошибок
5. Реализовать логирование в отдельный файл