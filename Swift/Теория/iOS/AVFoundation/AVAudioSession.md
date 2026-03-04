#avaudiosession #avfoundation #audio #ios #swift #background-audio #category #mode #route #interruption #metering #ios-10+ #concurrency

---
**(аудиосессия / сессия аудио)**

**AVAudioSession** — это **единственный и обязательный** объект в [[iOS]]/macOS, который управляет **всем аудио-контекстом** приложения: 

- какую категорию аудио использовать (воспроизведение, запись, VoIP, игра и т.д.)
- как вести себя при прерывании (звонок, Siri, другой плеер)
- может ли приложение играть звук в фоне
- какой маршрут вывода звука (динамик, наушники, Bluetooth, AirPlay)
- уровень громкости, ducking (приглушение), mixing (смешивание с другими приложениями)

Без правильной настройки `AVAudioSession` ваше приложение либо **не будет играть звук**, либо **будет вести себя некорректно** при звонках, Siri, AirPlay, фоновом режиме и т.д.

В 2026 году это **самый критический** компонент любого приложения с аудио (музыка, подкасты, голосовые сообщения, игры, рингтоны, уведомления, VoIP, AR/VR).

### 1. Основные категории (AVAudioSession.Category)

| Категория (iOS 10+)                       | Когда использовать                                           | Фоновая работа? | Микширование? | Самый частый сценарий 2026 |
|-------------------------------------------|--------------------------------------------------------------|------------------|---------------|-----------------------------|
| `.playback`                               | Музыка, подкасты, аудиокниги, видео без микрофона           | Да               | Нет (по умолчанию) | Музыкальные плееры, YouTube-подобные |
| `.playAndRecord`                          | Голосовые звонки, VoIP, запись + воспроизведение             | Да (с флагом)    | Да (по умолчанию) | WhatsApp, Telegram, Zoom, FaceTime |
| `.ambient`                                | Звуковые эффекты, которые можно приглушить (игры, уведомления) | Нет              | Да            | Игры, уведомления, фоновые звуки |
| `.soloAmbient`                            | То же, но без микширования (заменяет всё остальное)          | Нет              | Нет           | Редко, старые игры |
| `.record`                                 | Только запись (диктофон, голосовые заметки)                  | Да (с флагом)    | Нет           | Диктофон, распознавание речи |
| `.multiRoute`                             | Несколько аудиовыходов одновременно (редко)                 | Да               | Да            | Профессиональные аудио-приложения |
| `.spokenAudio` (iOS 13+)                  | Аудиокниги, подкасты, Siri, VoiceOver                        | Да               | Да            | Книги, подкасты с ducking |

**Рекомендация 2026:**  
- Музыка/видео → `.playback`
- Чат с голосом → `.playAndRecord`
- Звуки игры/уведомления → `.ambient`

### 2. Основные режимы (AVAudioSession.Mode)

| Режим                                     | Когда использовать                                           | Самый частый сценарий |
|-------------------------------------------|--------------------------------------------------------------|-----------------------|
| `.default`                                | Обычное воспроизведение/запись                               | Почти всегда по умолчанию |
| `.voiceChat`                              | VoIP, звонки (оптимизация для речи)                          | Telegram, WhatsApp |
| `.videoChat`                              | Видеозвонки (FaceTime, Zoom)                                 | Zoom-подобные |
| `.videoRecording`                         | Запись видео с микрофоном                                    | Камера, TikTok |
| `.measurement`                            | Измерение громкости (без обработки)                          | Микрофонные тесты |
| `.gameChat`                               | Игровой голосовой чат                                        | Fortnite, Call of Duty |
| `.moviePlayback` (iOS 13+)                | Кино/видео с оптимизацией громкости                          | Netflix-подобные |

### 3. Самый популярный и рекомендуемый паттерн 2026 года  
(Настройка сессии + обработка прерываний + фоновая работа)

```swift
import UIKit
import AVFoundation

class AudioManager {
    static let shared = AudioManager()
    
    private init() {
        setupSession()
        setupNotifications()
    }
    
    private func setupSession() {
        let session = AVAudioSession.sharedInstance()
        
        do {
            // 1. Категория и режим
            try session.setCategory(.playback, mode: .default, options: [.duckOthers, .allowBluetooth])
            // .duckOthers — приглушает другие приложения
            // .allowBluetooth — поддержка наушников
            
            // 2. Активируем сессию
            try session.setActive(true)
            
            print("Аудиосессия настроена: \(session.category)")
        } catch {
            print("Ошибка настройки AVAudioSession: \(error)")
        }
    }
    
    private func setupNotifications() {
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(handleInterruption),
            name: AVAudioSession.interruptionNotification,
            object: nil
        )
        
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(handleRouteChange),
            name: AVAudioSession.routeChangeNotification,
            object: nil
        )
    }
    
    @objc private func handleInterruption(_ notification: Notification) {
        guard let info = notification.userInfo,
              let typeValue = info[AVAudioSessionInterruptionTypeKey] as? UInt,
              let type = AVAudioSession.InterruptionType(rawValue: typeValue) else {
            return
        }
        
        switch type {
        case .began:
            // Звонок, Siri, другое приложение → пауза
            pausePlayback()
            
        case .ended:
            guard let optionsValue = info[AVAudioSessionInterruptionOptionKey] as? UInt else { return }
            let options = AVAudioSession.InterruptionOptions(rawValue: optionsValue)
            
            if options.contains(.shouldResume) {
                // Можно возобновить
                resumePlayback()
            }
        @unknown default:
            break
        }
    }
    
    @objc private func handleRouteChange(_ notification: Notification) {
        guard let info = notification.userInfo,
              let reasonValue = info[AVAudioSessionRouteChangeReasonKey] as? UInt,
              let reason = AVAudioSession.RouteChangeReason(rawValue: reasonValue) else {
            return
        }
        
        switch reason {
        case .oldDeviceUnavailable:
            // Наушники отключены → пауза или переключение на динамик
            pausePlayback()
        default:
            break
        }
    }
    
    func pausePlayback() {
        // Ваш код паузы плеера (AVAudioPlayer, AVPlayer и т.д.)
    }
    
    func resumePlayback() {
        // Возобновление
    }
    
    deinit {
        NotificationCenter.default.removeObserver(self)
    }
}
```

### Лучшие практики AVAudioSession в 2026 году

- **Настраивайте сессию один раз** — обычно в [[AppDelegate]] / [[SceneDelegate]] или singleton (как выше)
- **Категория + режим + options** — выбирайте в зависимости от сценария
- **Обработка прерываний** — обязательно подписывайтесь на `.interruptionNotification`
- **Фоновая работа** — добавьте `UIBackgroundModes` → `audio` в Info.plist + `.playback` категория
- **Duck / mix** — используйте `.duckOthers` для приглушения других приложений
- **Bluetooth / AirPlay** — добавляйте `.allowBluetooth`, `.allowAirPlay`
- **Для [[SwiftUI]]** — настройка в `UIApplicationDelegate` или через `AVAudioSession.sharedInstance()`
- **Документируйте** — пишите комментарий:

```swift
/// Настройка AVAudioSession для фонового воспроизведения музыки
private func setupAudioSession() {
    // ...
}
```

**Короткий итог 2026**:
> **AVAudioSession** — **единственный менеджер** всего аудио в приложении: категория, режим, прерывания, фон, маршрут.  
> Junior: "Говорит iPhone, как играть звук и что делать при звонке".  
> Middle: `.playback` для музыки, `.playAndRecord` для VoIP, обработка `.interruptionNotification`.  
> Senior: настройка один раз, `duckOthers`, `allowBluetooth`, фон через `UIBackgroundModes`, тестирование прерываний (звонок, Siri, AirPlay).  
