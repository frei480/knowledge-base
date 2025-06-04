---
tags:
  - csharp
---
Чтобы отслеживать прогресс выполнения задач в WPF на C#, можно использовать асинхронное программирование с `async/await` и механизмы отчетов о прогрессе. Вот пример реализации:

**1. XAML (окно с ProgressBar):**
```xml
<Window x:Class="ProgressExample.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Task Progress" Height="200" Width="300">
    <StackPanel Margin="10">
        <ProgressBar Minimum="0" Maximum="100" 
                     Height="20" Value="{Binding ProgressValue}"/>
        <TextBlock Text="{Binding ProgressValue, StringFormat=Progress: {0}%}" 
                   HorizontalAlignment="Center"/>
        <Button Content="Start Tasks" Command="{Binding StartTasksCommand}" 
                Margin="0 10"/>
    </StackPanel>
</Window>
```

**2. ViewModel с реализацией:**
```csharp
using System.Collections.ObjectModel;
using System.ComponentModel;
using System.Runtime.CompilerServices;
using System.Windows.Input;
using System.Threading.Tasks;

public class MainViewModel : INotifyPropertyChanged
{
    private int _progressValue;
    public int ProgressValue
    {
        get => _progressValue;
        set { _progressValue = value; OnPropertyChanged(); }
    }

    public ICommand StartTasksCommand => new RelayCommand(async () => await RunTasks());

    private async Task RunTasks()
    {
        ProgressValue = 0;
        var tasks = new List<Task>();
        var progress = new Progress<int>(ReportProgress);
        
        // Создаем несколько задач
        for (int i = 0; i < 5; i++)
        {
            tasks.Add(Task.Run(() => LongRunningTask(progress)));
        }

        await Task.WhenAll(tasks);
    }

    private void ReportProgress(int value)
    {
        // Обновление прогресса с учетом всех задач
        ProgressValue += value / 5; // Для 5 задач
    }

    private void LongRunningTask(IProgress<int> progress)
    {
        for (int i = 0; i <= 100; i++)
        {
            Task.Delay(50).Wait(); // Имитация работы
            progress.Report(1);   // Сообщаем о 1% прогрессе для этой задачи
        }
    }

    public event PropertyChangedEventHandler? PropertyChanged;
    protected virtual void OnPropertyChanged([CallerMemberName] string? propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}

public class RelayCommand : ICommand
{
    private readonly Func<Task> _execute;
    
    public RelayCommand(Func<Task> execute) => _execute = execute;
    
    public bool CanExecute(object? parameter) => true;
    
    public async void Execute(object? parameter) => await _execute();
    
    public event EventHandler? CanExecuteChanged;
}
```

**3. Основные моменты реализации:**

1. **Механизм отчетов (IProgress<T>):**
   - `Progress<int>` автоматически маршалит вызовы в UI-поток
   - Метод `ReportProgress` агрегирует прогресс от всех задач

2. **Многопоточность:**
   - Задачи выполняются в пуле потоков через `Task.Run`
   - `Task.WhenAll` ожидает завершения всех задач

3. **Обновление UI:**
   - Привязка данных через `INotifyPropertyChanged`
   - Прогресс-бар автоматически обновляется через Binding

**4. Дополнительные улучшения:**
- Добавьте отмену через `CancellationToken`
- Реализуйте детальный прогресс для каждой задачи
- Добавьте обработку ошибок в задачах
- Используйте `SemaphoreSlim` для ограничения параллелизма

**5. Примечания:**
- Для сложных сценариев рассмотрите использование `System.Reactive` (Rx.NET)
- Для параллельной обработки данных используйте `Parallel.ForEach` с отчетом о прогрессе
- Для длительных операций в UI-потоке используйте `Dispatcher.PushFrame`

Этот пример показывает базовую реализацию отслеживания прогресса для группы задач. Вы можете модифицировать его в зависимости от конкретных требований вашего приложения.