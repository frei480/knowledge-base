---
tags:
  - csharp
---

Для обновления ProgressBar в WPF с использованием паттерна MVVM из другого потока необходимо учитывать следующие аспекты:

# 1. Привязка Данных В XAML
Убедитесь, что ProgressBar привязан к свойству ViewModel:
```xml
<ProgressBar Minimum="0" Maximum="100" Value="{Binding ProgressValue}" />
```

# 2. Реализация ViewModel
Используйте `INotifyPropertyChanged` для уведомления об изменениях свойства.

## 2.1. Использование `IProgress<T>` (рекомендуется Для Асинхронных операций)
```csharp
using System.ComponentModel;
using System.Runtime.CompilerServices;
using System.Threading.Tasks;

public class MyViewModel : INotifyPropertyChanged
{
    private int _progressValue;
    public int ProgressValue
    {
        get => _progressValue;
        set
        {
            _progressValue = value;
            OnPropertyChanged();
        }
    }

    public event PropertyChangedEventHandler? PropertyChanged;

    protected virtual void OnPropertyChanged([CallerMemberName] string? propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }

    public async Task StartLongOperationAsync()
    {
        var progress = new Progress<int>(value => ProgressValue = value);
        await Task.Run(() => PerformWork(progress));
    }

    private void PerformWork(IProgress<int> progress)
    {
        for (int i = 0; i <= 100; i++)
        {
            // Имитация работы
            Thread.Sleep(50);
            progress.Report(i);
        }
    }
}
```

## 2.2. Использование `SynchronizationContext` (для Ручного Управления потоками)
```csharp
using System.ComponentModel;
using System.Runtime.CompilerServices;
using System.Threading;

public class MyViewModel : INotifyPropertyChanged
{
    private readonly SynchronizationContext _syncContext;
    private int _progressValue;

    public MyViewModel()
    {
        // Захватываем контекст синхронизации UI-потока
        _syncContext = SynchronizationContext.Current;
    }

    public int ProgressValue
    {
        get => _progressValue;
        set
        {
            if (_syncContext != null && _syncContext != SynchronizationContext.Current)
            {
                _syncContext.Post(_ => 
                {
                    _progressValue = value;
                    OnPropertyChanged();
                }, null);
            }
            else
            {
                _progressValue = value;
                OnPropertyChanged();
            }
        }
    }

    // Остальная реализация INotifyPropertyChanged аналогична предыдущему примеру
}
```

# 3. Запуск Операции
```csharp
// В команде или другом методе ViewModel
await StartLongOperationAsync();

// Или для SynchronizationContext
Task.Run(() => 
{
    for (int i = 0; i <= 100; i++)
    {
        Thread.Sleep(50);
        ProgressValue = i;
    }
});
```

# Основные Моменты:
1. **`IProgress<T>`** автоматически маршалит вызовы в UI-поток, если создан в нём.
2. **`SynchronizationContext`** позволяет вручную управлять потоком выполнения.
3. Всегда проверяйте контекст синхронизации при работе с потоками.

Эти подходы сохраняют концепции MVVM и обеспечивают потокобезопасность.