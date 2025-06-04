---
tags:
  - csharp
---
Чтобы передать значение из ViewModel в CommandParameter в WPF, вы можете использовать привязку данных. Ваша ViewModel должна содержать свойство, значение которого вы хотите передать в качестве параметра команды. Затем вы можете использовать это свойство в CommandParameter в XAML.
▎Пример
1. Создайте ViewModel:
```cs
using System.ComponentModel;
using System.Windows.Input;

public class MainViewModel : INotifyPropertyChanged
{
    private string _message;

    public string Message
    {
        get => _message;
        set
        {
            _message = value;
            OnPropertyChanged(nameof(Message));
        }
    }

    public ICommand ButtonCommand { get; }

    public MainViewModel()
    {
        Message = "Hello from ViewModel!";
        ButtonCommand = new RelayCommand(ExecuteButtonCommand);
    }

    private void ExecuteButtonCommand(object parameter)
    {
        // parameter будет равно значению свойства Message
        string message = parameter as string;
        MessageBox.Show(message);
    }

    public event PropertyChangedEventHandler PropertyChanged;

    protected void OnPropertyChanged(string propertyName)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```

2. Создайте RelayCommand (если у вас его еще нет):

```cs
using System;
using System.Windows.Input;

public class RelayCommand : ICommand
{
    private readonly Action<object> _execute;
    private readonly Predicate<object> _canExecute;

    public RelayCommand(Action<object> execute, Predicate<object> canExecute = null)
    {
        _execute = execute ?? throw new ArgumentNullException(nameof(execute));
        _canExecute = canExecute;
    }

    public bool CanExecute(object parameter) => _canExecute == null || _canExecute(parameter);

    public void Execute(object parameter) => _execute(parameter);

    public event EventHandler CanExecuteChanged
    {
        add => CommandManager.RequerySuggested += value;
        remove => CommandManager.RequerySuggested -= value;
    }
}

```
3. Привязка в XAML:

Теперь в вашем XAML вы можете использовать привязку к свойству Message в качестве CommandParameter:

```xml
<Window x:Class="YourNamespace.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="MainWindow" Height="200" Width="300">
    <Window.DataContext>
        <local:MainViewModel />
    </Window.DataContext>

    <Grid>
        <Button Content="Нажми меня" 
                Width="100" 
                Height="30" 
                Command="{Binding ButtonCommand}" 
                CommandParameter="{Binding Message}" 
                HorizontalAlignment="Center" 
                VerticalAlignment="Center"/>
    </Grid>
</Window>
```

▎Объяснение:
• ViewModel: Ваша ViewModel содержит свойство Message, которое будет передано в качестве параметра.
• RelayCommand: Это реализация интерфейса ICommand, которая позволяет вам выполнять команды.
• XAML: В CommandParameter используется привязка к свойству Message вашей ViewModel. Когда кнопка нажата, метод ExecuteButtonCommand будет вызван с параметром, равным текущему значению свойства Message.
Таким образом, вы можете легко передавать значения из вашей ViewModel в качестве параметров команд в WPF.