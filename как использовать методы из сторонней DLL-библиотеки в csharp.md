---
tags:
  - csharp
---
Чтобы использовать методы из сторонней DLL-библиотеки в C#, существует два основных подхода в зависимости от типа библиотеки:

---

# 1. **Для Управляемых DLL (написанных На .NET)**
Если DLL создана на C# или другом языке .NET:
1. **Добавьте ссылку на DLL:**
   - В Visual Studio: правой кнопкой по проекту → **Add → Reference → Browse** → выберите DLL.
   - Через .csproj:
     ```xml
     <ItemGroup>
       <Reference Include="YourLibrary">
         <HintPath>Путь\к\YourLibrary.dll</HintPath>
       </Reference>
     </ItemGroup>
     ```

2. **Используйте классы и методы:**
   ```csharp
   using YourLibrary.Namespace;

   public class Program
   {
       public static void Main()
       {
           var obj = new ClassFromDLL();
           obj.MethodFromDLL();
       }
   }
   ```

---

# 2. **Для Нативных DLL (написанных На C/C++ И т.д.)**
Используйте **Platform Invocation Services (P/Invoke)**:

## A. **Объявите Методы DLL**
Используйте атрибут `DllImport`:
```csharp
using System.Runtime.InteropServices;

public class NativeWrapper
{
    // Пример объявления функции из DLL
    [DllImport("YourNativeLibrary.dll", CharSet = CharSet.Auto)]
    public static extern int YourNativeMethod(int param1, string param2);
}
```

## B. **Вызовите метод**
```csharp
public class Program
{
    public static void Main()
    {
        int result = NativeWrapper.YourNativeMethod(42, "text");
    }
}
```

---

# **Важные нюансы:**
1. **Расположение DLL:**
   - Поместите DLL в папку с исполняемым файлом (`bin\Debug` или `bin\Release`).
   - Либо укажите полный путь в `DllImport`.

2. **Соглашение о вызовах:**
   - Если в DLL используется `__stdcall`, укажите это:
     ```csharp
     [DllImport("YourDLL.dll", CallingConvention = CallingConvention.StdCall)]
     ```

2. **Маршалинг данных:[[маршалинг данных]]**
   - Для сложных типов (структуры, указатели) используйте `MarshalAs`:
     ```csharp
     [DllImport("YourDLL.dll")]
     public static extern void ReadData(
         [MarshalAs(UnmanagedType.LPStr)] string input,
         out IntPtr output
     );
     ```

4. **Исключения:**
   - Обрабатывайте ошибки через `try-catch` при вызове нативных методов.

---

# **Советы:**
- **Проверьте разрядность DLL:** x86/x64 должна совпадать с вашим проектом.
- **Используйте утилиты:**
  - **Dependency Walker** (для просмотра экспортируемых функций).
  - **dumpbin** (через Developer Command Prompt: `dumpbin /exports YourDLL.dll`).

---

**Пример полного кода для нативной DLL:**
```csharp
using System;
using System.Runtime.InteropServices;

public class Program
{
    [DllImport("User32.dll", CharSet = CharSet.Unicode)]
    public static extern int MessageBox(IntPtr hWnd, string text, string caption, uint type);

    static void Main()
    {
        MessageBox(IntPtr.Zero, "Hello from DLL!", "Message", 0);
    }
}
```

Если вы столкнулись с ошибкой `DllNotFoundException`, проверьте путь к DLL и разрядность проекта.