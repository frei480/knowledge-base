---
tags:
  - АрхитектураПО
---

# Паттерн Facade (Фасад)
**Назначение:**
Предоставляет унифицированный интерфейс к сложной подсистеме, скрывая её детали реализации. Упрощает взаимодействие клиента с системой, уменьшая зависимости.

## Ключевые Аспекты:
1. **Упрощение интерфейса:** Инкапсулирует сложные операции подсистемы.
2. **Сокрытие сложности:** Клиент работает только с Фасадом, не зная о внутренних компонентах.
3. **Снижение связанности:** Клиент зависит только от Фасада, а не от множества классов подсистемы.

---

# Пример На Python
**Сценарий:** Система обработки видеофайлов (аудио-декодирование, видео-декодирование, конвертация).

```python
# Подсистема: Аудио-декодер
class AudioDecoder:
    def decode_audio(self, file):
        print(f"Аудио декодировано: {file}")
        return f"DECODED_AUDIO:{file}"

# Подсистема: Видео-декодер
class VideoDecoder:
    def decode_video(self, file):
        print(f"Видео декодировано: {file}")
        return f"DECODED_VIDEO:{file}"

# Подсистема: Конвертер
class VideoConverter:
    def convert(self, audio, video, format):
        print(f"Конвертация в {format}: [аудио: {audio}, видео: {video}]")
        return f"Конвертированный файл.{format}"

# Фасад
class VideoProcessingFacade:
    def __init__(self):
        self.audio_decoder = AudioDecoder()
        self.video_decoder = VideoDecoder()
        self.converter = VideoConverter()
    
    def process_video(self, audio_file, video_file, output_format):
        audio = self.audio_decoder.decode_audio(audio_file)
        video = self.video_decoder.decode_video(video_file)
        return self.converter.convert(audio, video, output_format)

# Клиентский код
facade = VideoProcessingFacade()
result = facade.process_video("sound.wav", "video.avi", "mp4")
print(f"Результат: {result}")
```

**Вывод:**
```
Аудио декодировано: sound.wav
Видео декодировано: video.avi
Конвертация в mp4: [аудио: DECODED_AUDIO:sound.wav, видео: DECODED_VIDEO:video.avi]
Результат: Конвертированный файл.mp4
```

---

# Пример На C\#
**Сценарий:** Умный дом (освещение, аудиосистема, кондиционер).

```csharp
// Подсистема: Освещение
class LightingSystem {
    public void DimLights() => Console.WriteLine("Освещение: приглушено");
    public void TurnOn() => Console.WriteLine("Освещение: включено");
}

// Подсистема: Аудио
class AudioSystem {
    public void PlayMusic(string track) => 
        Console.WriteLine($"Аудио: воспроизводится '{track}'");
}

// Подсистема: Климат-контроль
class ClimateControl {
    public void SetTemperature(int temp) => 
        Console.WriteLine($"Кондиционер: установлено {temp}°C");
}

// Фасад
class SmartHomeFacade {
    private LightingSystem _lighting;
    private AudioSystem _audio;
    private ClimateControl _climate;

    public SmartHomeFacade() {
        _lighting = new LightingSystem();
        _audio = new AudioSystem();
        _climate = new ClimateControl();
    }

    public void StartEveningMode() {
        _lighting.DimLights();
        _audio.PlayMusic("Джаз");
        _climate.SetTemperature(22);
        Console.WriteLine("Режим 'Вечер': активирован\n");
    }
}

// Клиентский код
class Program {
    static void Main() {
        var home = new SmartHomeFacade();
        home.StartEveningMode();
    }
}
```

**Вывод:**
```
Освещение: приглушено
Аудио: воспроизводится 'Джаз'
Кондиционер: установлено 22°C
Режим 'Вечер': активирован
```

---

# Против Каких Антипаттернов Борется Facade?
1. **[[опасные антипаттерны в ООП#^285f2b|Spaghetti Code (Спагетти-код)]]**
   **Проблема:** Клиент напрямую управляет множеством классов подсистемы, создавая запутанные зависимости.
   **Решение:** Фасад инкапсулирует сложные взаимодействия, предоставляя чистый интерфейс.

2. **[[опасные антипаттерны в ООП#^19f14b|Tight Coupling (Жёсткая связанность)]]**
   **Проблема:** Клиент зависит от внутренних классов подсистемы, что усложняет изменения.
   **Решение:** Фасад ослабляет связанность — клиент зависит только от него.

3. **Large Interface (Избыточный интерфейс)**
   **Проблема:** Подсистема предоставляет слишком много методов, большинство из которых клиенту не нужны.
   **Решение:** Фасад предлагает специализированный минимум операций.

4. **Violation of Law of Demeter (Нарушение [[принцип наименьшего знания закон Деметры python и csharp|закона Деметры]])**
   **Проблема:** Клиент обращается к глубоким уровням вложенности объектов (`a.GetB().GetC().DoSomething()`).
   **Решение:** Фасад берёт на себя координацию вложенных вызовов.

---

# Когда Использовать Facade?
1. Для упрощения взаимодействия со сложной библиотекой или API.
2. При рефакторинге унаследованного кода с высокой связанностью.
3. Для создания «точки входа» в слоистую архитектуру (например, сервисный слой в приложении).
4. Чтобы изолировать клиентский код от изменений в подсистеме.

> **Важно:** Фасад не запрещает прямой доступ к подсистеме! Он предлагает *альтернативный*, упрощённый интерфейс, но при необходимости клиент может использовать классы подсистемы напрямую.