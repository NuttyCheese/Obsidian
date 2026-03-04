#avaudioplayer #audio #avfoundation #ios #swift #playback #sound #music #delegate #background-audio #metering #seek #rate #volume

---
**(плеер аудио / аудиоплеер)**

**AVAudioPlayer** — это основной класс из фреймворка **[[AVFoundation]]** для **воспроизведения** аудиофайлов в [[iOS]]/macOS-приложениях.

Он предназначен для **простого и надёжного** проигрывания локальных аудиофайлов (mp3, m4a, wav, caf, aac и др.) с поддержкой:

- воспроизведения / паузы / остановки
- перемотки (seek)
- изменения громкости и скорости
- зацикливания (loop)
- измерения уровня громкости (metering)
- фона (background playback)
- нескольких одновременно звучащих файлов (микширование)

В 2026 году это **самый популярный и рекомендуемый** способ воспроизведения локального аудио в [[UIKit]]-приложениях (подкасты, музыка, звуковые эффекты, голосовые сообщения, рингтоны и т.д.).

### Когда использовать AVAudioPlayer в 2026 году

| Сценарий                                      | Почему именно AVAudioPlayer                                 | Альтернатива |
|-----------------------------------------------|-------------------------------------------------------------|--------------|
| Воспроизведение локальных mp3/m4a файлов      | Простота, надёжность, низкий overhead                      | AVAudioEngine (если нужен сложный микс) |
| Звуковые эффекты (клик, уведомление, взрыв)   | Мгновенный запуск, несколько экземпляров одновременно       | SystemSoundID (короткие <5 сек) |
| Подкасты / аудиокниги                         | Поддержка seek, duration, metering                          | AVPlayer (для стриминга) |
| Фоновая музыка в приложении                   | Работает в фоне с правильной настройкой сессии              | AVPlayer + AVAudioSession |
| Голосовые сообщения в чате                    | Локальное воспроизведение + metering (визуализация громкости) | — |

### Основные свойства и методы AVAudioPlayer

| Свойство / Метод                                       | Тип / Возвращает               | Что делает / зачем нужно                    | Самый частый сценарий         |
| ------------------------------------------------------ | ------------------------------ | ------------------------------------------- | ----------------------------- |
| `init(contentsOf:)` / `init(data:)`                    | `AVAudioPlayer?`               | Создание плеера из файла / данных           | Основной инициализатор        |
| `play()` / `pause()` / `stop()`                        | `Bool` (успех)                 | Управление воспроизведением                 | Основные кнопки плеера        |
| `isPlaying`                                            | [[Bool]]                       | Проверяет, играет ли сейчас                 | Обновление UI (play/pause)    |
| `currentTime`                                          | [[TimeInterval]]               | Текущее время воспроизведения (секунды)     | Seek, прогресс-бар            |
| `duration`                                             | `TimeInterval`                 | Общая длительность трека                    | Показ длительности            |
| `rate`                                                 | `Float` (0.5–2.0, default 1.0) | Скорость воспроизведения                    | 0.5x–2x для подкастов         |
| `volume`                                               | [[Float]] (0.0–1.0)            | Громкость (относительно системной)          | Ползунок громкости            |
| `numberOfLoops`                                        | [[Int]] (-1 = бесконечно)      | Зацикливание                                | Фоновая музыка                |
| `enableRate`                                           | `Bool` (false)                 | Включает изменение скорости                 | Обязательно для rate ≠ 1.0    |
| `meteringEnabled`                                      | `Bool` (false)                 | Включает измерение уровня громкости         | Визуализация громкости        |
| `updateMeters()`                                       | `Void`                         | Обновляет значения metering                 | Вызывать каждые 0.1 сек       |
| `peakPower(forChannel:)` / `averagePower(forChannel:)` | `Float` (-160…0 dB)            | Уровень громкости в канале                  | Анимация громкости            |
| `delegate`                                             | [[AVAudioPlayerDelegate]]`?`   | Обработка завершения, ошибок                | `audioPlayerDidFinishPlaying` |
| `prepareToPlay()`                                      | `Bool`                         | Подготавливает буфер (ускоряет первый play) | Перед первым воспроизведением |

### Самый популярный паттерн 2026 года  
(AVAudioPlayer + [[AVAudioSession]] + Delegate + [[Combine]] + прогресс)

```swift
import UIKit
import AVFoundation
import Combine

class AudioPlayerViewController: UIViewController, AVAudioPlayerDelegate {
    
    private var player: AVAudioPlayer?
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var playButton: UIButton = {
        let btn = UIButton(type: .system)
        btn.setTitle("Play", for: .normal)
        btn.addTarget(self, action: #selector(togglePlayback), for: .touchUpInside)
        btn.translatesAutoresizingMaskIntoConstraints = false
        return btn
    }()
    
    private lazy var progressView: UIProgressView = {
        let pv = UIProgressView()
        pv.progress = 0
        pv.translatesAutoresizingMaskIntoConstraints = false
        return pv
    }()
    
    private lazy var timeLabel: UILabel = {
        let lbl = UILabel()
        lbl.text = "00:00 / 00:00"
        lbl.textAlignment = .center
        lbl.translatesAutoresizingMaskIntoConstraints = false
        return lbl
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        
        let stack = UIStackView(arrangedSubviews: [playButton, progressView, timeLabel])
        stack.axis = .vertical
        stack.spacing = 20
        stack.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(stack)
        
        NSLayoutConstraint.activate([
            stack.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            stack.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            stack.widthAnchor.constraint(equalToConstant: 300)
        ])
        
        setupAudioSession()
        loadAudioFile()
    }
    
    private func setupAudioSession() {
        do {
            let session = AVAudioSession.sharedInstance()
            try session.setCategory(.playback, mode: .default)
            try session.setActive(true)
        } catch {
            print("Ошибка настройки аудиосессии: \(error)")
        }
    }
    
    private func loadAudioFile() {
        guard let url = Bundle.main.url(forResource: "sample", withExtension: "mp3") else { return }
        
        do {
            player = try AVAudioPlayer(contentsOf: url)
            player?.delegate = self
            player?.prepareToPlay()
            player?.enableRate = true  // для изменения скорости
            updateTimeLabel()
        } catch {
            print("Ошибка загрузки аудио: \(error)")
        }
    }
    
    @objc private func togglePlayback() {
        guard let player else { return }
        
        if player.isPlaying {
            player.pause()
            playButton.setTitle("Play", for: .normal)
        } else {
            player.play()
            playButton.setTitle("Pause", for: .normal)
            startProgressTimer()
        }
    }
    
    private func startProgressTimer() {
        Timer.scheduledTimer(withTimeInterval: 0.1, repeats: true) { [weak self] timer in
            guard let self, let player = self.player else {
                timer.invalidate()
                return
            }
            
            if !player.isPlaying {
                timer.invalidate()
                return
            }
            
            let progress = Float(player.currentTime / player.duration)
            self.progressView.progress = progress
            
            let current = Int(player.currentTime)
            let total = Int(player.duration)
            self.timeLabel.text = "\(String(format: "%02d:%02d", current/60, current%60)) / \(String(format: "%02d:%02d", total/60, total%60))"
        }
    }
    
    private func updateTimeLabel() {
        guard let player else { return }
        let total = Int(player.duration)
        timeLabel.text = "00:00 / \(String(format: "%02d:%02d", total/60, total%60))"
    }
    
    // MARK: - AVAudioPlayerDelegate
    
    func audioPlayerDidFinishPlaying(_ player: AVAudioPlayer, successfully flag: Bool) {
        playButton.setTitle("Play", for: .normal)
        progressView.progress = 0
        updateTimeLabel()
    }
    
    func audioPlayerDecodeErrorDidOccur(_ player: AVAudioPlayer, error: Error?) {
        print("Ошибка воспроизведения: \(error?.localizedDescription ?? "неизвестно")")
    }
}
```

### Лучшие практики AVAudioPlayer в 2026 году

- **AVAudioSession** — всегда настраивайте категорию `.playback` для фонового воспроизведения  
- **prepareToPlay()** — вызывайте перед первым `play()` для снижения задержки  
- **enableRate = true** — если нужна скорость воспроизведения  
- **meteringEnabled = true + updateMeters()** — для визуализации громкости  
- **delegate** — обязательно реализуйте `audioPlayerDidFinishPlaying`  
- **background playback** — добавьте `UIBackgroundModes` → `audio` в Info.plist  
- **Combine / async** — используйте `Timer.publish` или `CADisplayLink` для прогресса  
- **Для SwiftUI** — оборачивайте в `UIViewControllerRepresentable` или используйте `AVPlayer`  
- **Документируйте** — пишите комментарий:

```swift
/// AVAudioPlayer для локального воспроизведения аудиофайлов с прогрессом и контролем
private var player: AVAudioPlayer?
```

**Короткий итог 2026**:
> **AVAudioPlayer** — **классический** и **самый надёжный** плеер для локальных аудиофайлов в iOS.  
> В 2026 году:  
> - ключевые методы — `play()`, `pause()`, `currentTime`, `duration`, `prepareToPlay()`  
> - самый популярный паттерн — `AVAudioPlayer` + `AVAudioSession` + delegate + прогресс через Timer  
> - идеален для звуковых эффектов, подкастов, голосовых сообщений, музыки  
> - для стриминга и сложного микса — используйте `AVPlayer` / `AVAudioEngine`  
> - это **основа** любого аудио в UIKit-приложениях  
