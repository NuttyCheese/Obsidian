**Core Audio** — это фундаментальный низкоуровневый фреймворк Apple для работы с **аудио** на всех платформах: iOS, iPadOS, macOS, watchOS, tvOS и visionOS.

По состоянию на 2026 год Core Audio остаётся **самым мощным и гибким** инструментом для задач, где требуется:

- максимальный контроль над аудиопотоком  
- минимальная задержка (low-latency)  
- обработка в реальном времени  
- многоканальный звук  
- работа с сырыми аудиобуферами

### Основные подфреймворки Core Audio (актуальные в 2026)

| Подфреймворк / API                  | Основное назначение                                      | Когда использовать в 2026 году                          | Современная альтернатива / примечание |
|-------------------------------------|----------------------------------------------------------|----------------------------------------------------------|----------------------------------------|
| **Audio Unit** (v3)                 | Плагины, эффекты, синтезаторы, генераторы, микшеры       | Кастомные AUv3-плагины, межприложенческий аудио-роутинг  | Audio Unit v3 — основной стандарт |
| **AVAudioEngine**                   | Высокоуровневая абстракция над Audio Unit                | Эквалайзер, реверб, запись/микширование в реал-тайм     | Самый популярный современный API |
| **Audio Queue**                     | Низкоуровневая запись/воспроизведение буферов           | Legacy-приложения, очень низкая задержка                | Почти полностью заменён AVAudioEngine |
| **Audio Converter**                 | Конвертация форматов (PCM ↔ compressed, sample rate)    | Транскодирование, ресэмплинг                            | Используется внутри AVFoundation |
| **Audio File**                      | Чтение/запись аудиофайлов (CAF, WAV, AAC, MP3 и т.д.)   | Работа с файлами без AVFoundation                       | Редко, чаще AVAudioFile |
| **Core Audio Types**                | Базовые структуры: AudioBufferList, AudioStreamBasicDescription | Низкоуровневая работа с буферами                        | Всё ещё основа |
| **AUGraph** (deprecated)            | Граф Audio Unit (старый API)                             | Никогда (удалён в iOS 15+, macOS 12+)                   | Полностью deprecated |

### Когда использовать Core Audio в 2026 году (реальные сценарии)

| Сценарий                                      | Рекомендуемый API                  | Почему именно Core Audio / AVAudioEngine |
|-----------------------------------------------|-------------------------------------|-------------------------------------------|
| Профессиональный аудио-редактор / DAW         | Audio Unit v3 + AVAudioEngine       | Максимальный контроль, низкая задержка    |
| Караоке / вокальный процессор                 | AVAudioEngine + AVAudioUnitEQ       | Эффекты в реальном времени                |
| Живое микширование нескольких источников      | AVAudioEngine + AVAudioMixerNode    | Гибкое маршрутирование                    |
| Синтезатор / MIDI-инструмент                  | Audio Unit v3 + AUAudioUnit         | Поддержка AUv3-плагина                    |
| Игровой звук (3D-аудио, пространственный)     | AVAudioEngine + AVAudioPlayerNode   | Достаточно для большинства игр            |
| Простое воспроизведение музыки / подкастов    | AVPlayer / AVAudioPlayer            | Не нужен Core Audio                       |
| Запись голосовых заметок / диктофон           | AVAudioRecorder                     | Достаточно                                |

### Самый популярный и рекомендуемый паттерн 2026 — AVAudioEngine

```swift
import AVFoundation

final class AudioProcessor {
    
    private let engine = AVAudioEngine()
    private let playerNode = AVAudioPlayerNode()
    private let eqNode = AVAudioUnitEQ(numberOfBands: 3)
    
    init() {
        // Подключаем ноды
        engine.attach(playerNode)
        engine.attach(eqNode)
        
        // Соединяем: player → EQ → main mixer
        let format = playerNode.outputFormat(forBus: 0)
        engine.connect(playerNode, to: eqNode, format: format)
        engine.connect(eqNode, to: engine.mainMixerNode, format: format)
        
        // Настраиваем EQ (пример: low-shelf + parametric + high-shelf)
        eqNode.bands[0].filterType = .lowShelf
        eqNode.bands[0].frequency = 120
        eqNode.bands[0].gain = 6
        eqNode.bands[0].bypass = false
        
        eqNode.bands[1].filterType = .parametric
        eqNode.bands[1].frequency = 1000
        eqNode.bands[1].gain = -3
        eqNode.bands[1].bandwidth = 1.0
        
        eqNode.bands[2].filterType = .highShelf
        eqNode.bands[2].frequency = 8000
        eqNode.bands[2].gain = 4
    }
    
    func play(fileURL: URL) throws {
        let file = try AVAudioFile(forReading: fileURL)
        playerNode.scheduleFile(file, at: nil)
        
        try engine.start()
        playerNode.play()
    }
    
    func stop() {
        playerNode.stop()
        engine.stop()
    }
    
    func setEQGain(band: Int, gain: Float) {
        eqNode.bands[band].gain = gain
    }
}
```

### Лучшие практики AVFoundation / Core Audio в Swift 2026

- **AVAudioEngine** — основной инструмент для большинства задач (заменил AUGraph и Audio Queue)  
- **actor** — для хранения состояния аудио-движка и буферов  
- **AVAudioSession** — обязательно настраивать категорию и режим (`.playAndRecord`, `.defaultToSpeaker`)  
- **Разрешения** — запрашивай микрофон и аудио-вывод асинхронно  
- **Реал-тайм обработка** — используй `AVAudioNode.installTap` или `AVAudioPlayerNode.scheduleBuffer`  
- **Swift 6 strict concurrency** — весь UI и сессия — `@MainActor`, обработка буферов — dedicated actor или Task.detached  
- **Тестирование** — сложно → чаще используют manual-тесты или Audio Unit Host для отладки  
- **Документируйте** — пишите комментарий «AVAudioEngine — обработка аудио в реальном времени»

**Короткий девиз 2026**:
> «AVFoundation / Core Audio — это когда тебе нужен полный контроль над аудио на уровне буферов и эффектов.  
> В 2026 году **AVAudioEngine** — основной инструмент для профессионального аудио.  
> Для простого воспроизведения/записи — AVPlayer / AVAudioPlayer / AVAudioRecorder.»

Удачи с мощной обработкой звука в Swift! 🎵