---
tags:
  - powershell
---

Для проверки наличия большого количества файлов на сетевом диске с помощью PowerShell можно создать скрипт, который читает пути из текстового файла и проверяет их существование. Пример решения:

```powershell
# Проверка наличия файлов из списка
$inputFile = "C:\path\to\your\file_list.txt"  # Файл со списком путей
$outputFile = "C:\path\to\your\results.csv"   # Файл для результатов
$networkPath = "X:\"                          # Базовый сетевой путь (если пути в файле относительные)

# Чтение списка файлов
$fileList = Get-Content -Path $inputFile

# Проверка каждого файла
$results = foreach ($file in $fileList) {
    $fullPath = if ([System.IO.Path]::IsPathRooted($file)) {
        $file
    } else {
        Join-Path -Path $networkPath -ChildPath $file
    }

    [PSCustomObject]@{
        FilePath = $fullPath
        Exists = Test-Path -Path $fullPath -PathType Leaf
    }
}

# Экспорт результатов
$results | Export-Csv -Path $outputFile -NoTypeInformation -Encoding UTF8

Write-Host "Проверено $($results.Count) файлов. Результаты сохранены в $outputFile"
```

### Как использовать:
1. Создайте текстовый файл со списком путей (пример содержимого):
   ```
   folder1\file1.txt
   folder2\image.jpg
   \full\path\on\network\document.docx
   ```

2. Измените переменные:
   - `$inputFile` - путь к вашему файлу со списком
   - `$outputFile` - путь для сохранения результатов
   - `$networkPath` - базовый путь, если используете относительные пути

3. Запустите скрипт от имени администратора (если нужны повышенные права)

### Особенности:
- Обрабатывает как абсолютные, так и относительные пути
- Возвращает результаты в CSV с полями `FilePath` и `Exists`
- Работает с большими файлами (проверка только существования)
- Автоматически добавляет базовый сетевой путь к относительным путям

### Оптимизация для очень больших списков:
```powershell
# Добавьте прогресс-бар
$fileList = Get-Content -Path $inputFile
$total = $fileList.Count
$counter = 0

$results = foreach ($file in $fileList) {
    $counter++
    Write-Progress -Activity "Проверка файлов" -Status "$counter из $total" -PercentComplete ($counter/$total*100)
    
    # ... остальной код проверки ...
}
```

### Важные примечания:
1. Используйте UNC-пути (`\\server\share\...`) вместо букв дисков (`X:\`), если есть проблемы с подключенными дисками
2. Убедитесь, что PowerShell имеет доступ к сетевой папке
3. Для ускорения можно добавить параллельную обработку, но это требует PowerShell 7+:
   ```powershell
   ForEach-Object -Parallel {
       # ... код проверки ...
   } -ThrottleLimit 10 -InputObject $fileList
   ```

Пример файла результатов в CSV:
```csv
"FilePath","Exists"
"X:\folder1\file1.txt","True"
"X:\folder2\image.jpg","False"
"\\server\share\document.docx","True"
```

[[PowerShell запуск проверки в многопоточном режиме]]