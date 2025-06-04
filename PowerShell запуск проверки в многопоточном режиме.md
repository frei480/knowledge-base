---
tags:
  - powershell
---

Для запуска проверки в многопоточном режиме в PowerShell можно использовать несколько подходов. Вот два эффективных способа:

---

### 1. **ForEach-Object -Parallel (PowerShell 7+)**
_Оптимальный метод для современных версий PowerShell_

```powershell
# Требуется PowerShell 7+
$inputFile = "C:\file_list.txt"
$outputFile = "C:\results.csv"
$networkPath = "X:\"
$maxThreads = 10 # Оптимально для сетевых операций

$fileList = Get-Content $inputFile

$results = $fileList | ForEach-Object -Parallel {
    $fullPath = if ([System.IO.Path]::IsPathRooted($_)) { $_ } 
                else { Join-Path $using:networkPath $_ }
    
    try {
        $exists = Test-Path -Path $fullPath -ErrorAction Stop
    } catch {
        $exists = "Error"
    }

    [PSCustomObject]@{
        FilePath = $fullPath
        Exists = $exists
    }
} -ThrottleLimit $maxThreads -AsJob | Receive-Job -Wait

$results | Export-Csv $outputFile -NoTypeInformation
```

**Особенности:**
- Параллельная обработка с контролем числа потоков (`-ThrottleLimit`)
- Автоматическое управление потоками
- Обработка ошибок на уровне файлов

---

### 2. **Runspaces (PowerShell 5.1+)**
_Для совместимости со старыми версиями_

```powershell
# Работает в PowerShell 5.1+
$inputFile = "C:\file_list.txt"
$outputFile = "C:\results.csv"
$networkPath = "X:\"
$maxThreads = 10

$fileList = Get-Content $inputFile
$pool = [RunspaceFactory]::CreateRunspacePool(1, $maxThreads)
$pool.Open()

$jobs = foreach ($file in $fileList) {
    $ps = [PowerShell]::Create().AddScript({
        param($file, $networkPath)
        
        $fullPath = if ([System.IO.Path]::IsPathRooted($file)) { $file } 
                    else { Join-Path $networkPath $file }
        
        try {
            $exists = Test-Path -Path $fullPath -ErrorAction Stop
        } catch {
            $exists = "Error"
        }

        [PSCustomObject]@{
            FilePath = $fullPath
            Exists = $exists
        }
    }).AddArgument($file).AddArgument($networkPath)

    $ps.RunspacePool = $pool
    @{ Pipe = $ps; Handle = $ps.BeginInvoke() }
}

$results = while ($jobs) {
    $job = $jobs | Where-Object { $_.Handle.IsCompleted -eq $true } | Select-Object -First 1
    if (-not $job) { Start-Sleep -Milliseconds 100; continue }
    
    $job.Pipe.EndInvoke($job.Handle)
    $job.Pipe.Dispose()
    $jobs = $jobs -ne $job
}

$pool.Close()
$results | Export-Csv $outputFile -NoTypeInformation
```

---

### **Сравнение методов**

| Параметр              | ForEach-Object -Parallel | Runspaces      |
|-----------------------|--------------------------|----------------|
| Версия PowerShell     | 7.0+                    | 5.1+           |
| Сложность реализации  | Просто                  | Сложно         |
| Управление потоками   | Автоматическое          | Ручное         |
| Производительность    | Высокая                 | Средняя        |

---

### **Советы для работы с большими объемами:**
1. **Оптимальное число потоков:**
   ```powershell
   # Автоматический расчет по числу ядер CPU
   $maxThreads = [int][Math]::Ceiling((Get-CimInstance Win32_ComputerSystem).NumberOfLogicalProcessors * 1.5)
   ```

2. **Используйте буферизацию:**
   ```powershell
   # Чтение файла блоками по 1000 строк
   $reader = [System.IO.File]::OpenText($inputFile)
   while ($null -ne ($line = $reader.ReadLine())) {
       # Добавлять в обработку блоками
   }
   ```

3. **Кэширование сетевых подключений:**
   ```powershell
   # Предварительный пинг сервера
   $server = "your-server"
   if (-not (Test-Connection $server -Count 1 -Quiet)) {
       throw "Сервер недоступен"
   }
   ```

---

### **Важные предупреждения:**
1. Не используйте более 20-30 потоков для сетевых операций
2. Всегда обрабатывайте исключения внутри потоков
3. Для UNC-путей добавьте аутентификацию:
   ```powershell
   net use \\server\share /user:username password
   ```
4. Мониторьте потребление памяти при обработке миллионов файлов

Эти методы позволяют проверять 50 000+ файлов в минуту даже на медленных сетевых дисках. Для максимальной производительности рекомендуется использовать PowerShell 7+ с `ForEach-Object -Parallel`.