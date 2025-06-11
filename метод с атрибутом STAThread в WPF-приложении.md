---
tags:
  - csharp
---

Чтобы запустить метод с атрибутом `[STAThread]` в WPF-приложении, следуйте инструкциям ниже.

# 1. **Проверьте Точку Входа приложения**
В WPF метод `Main` автоматически генерируется с атрибутом `[STAThread]` в файле `App.g.cs`. Убедитесь, что вы не переопределили его вручную без этого атрибута. Пример корректной точки входа:

```csharp
using System.Windows;

public partial class App : Application
{
    [STAThread] // Обязательный атрибут для главного потока
    public static void Main()
    {
        App app = new App();
        app.InitializeComponent();
        app.Run();
    }
}
```

# 2. **Если Нужно Создать Отдельный STA-поток**
Если требуется выполнить код в другом STA-потоке (например, для работы с COM-объектами), создайте поток с `ApartmentState.STA`:

```csharp
using System.Threading;
using System.Windows;

public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
    }

    private void Button_Click(object sender, RoutedEventArgs e)
    {
        Thread staThread = new Thread(() =>
        {
            // Код, требующий STA (например, работа с буфером обмена)
            YourSTARequiredMethod();
        });

        staThread.SetApartmentState(ApartmentState.STA); // Установите STA для потока
        staThread.Start();
    }

    private void YourSTARequiredMethod()
    {
        // Ваша логика здесь
    }
}
```

# 3. **Важные замечания**
- **Главный поток WPF уже STA**: Не нужно явно указывать `[STAThread]` для других методов, кроме точки входа `Main`.
- **Работа с UI**: Элементы интерфейса WPF можно обновлять только в главном потоке. Используйте `Dispatcher.Invoke` для доступа к UI из других потоков:
  ```csharp
  Dispatcher.Invoke(() => 
  {
      textBox.Text = "Обновление из другого потока";
  });
  ```

# Пример Полной Настройки Точки Входа
Если вы вручную контролируете метод `Main`, добавьте атрибут `[STAThread]`:

```csharp
[STAThread]
public static void Main()
{
    Application app = new Application();
    MainWindow window = new MainWindow();
    app.Run(window);
}
```

Это гарантирует, что главный поток приложения будет STA.