#avaudioplayerdelegate #avfoundation #audio #ios #swift #delegate #playback #completion #error-handling #interruption #metering #background-audio

---
**(делегат аудиоплеера / обработчик событий AVAudioPlayer)**

**AVAudioPlayerDelegate** — это протокол в [[AVFoundation]], который позволяет вашему приложению **реагировать на ключевые события** воспроизведения аудио через `AVAudioPlayer`:

- завершение проигрывания трека
- ошибки декодирования
- прерывания (звонок, Siri, другое приложение)
- окончание времени воспроизведения в фоне

Это **единственный официальный способ** узнать, когда плеер закончил играть, произошла ошибка или сессия была прервана.

### Основные методы протокола (2026 актуально)

| Метод делегата                                      | Когда вызывается                                                                 | Что обычно делают внутри метода                                      | Самый частый сценарий |
|-----------------------------------------------------|----------------------------------------------------------------------------------|-----------------------------------------------------------------------|-----------------------|
| `audioPlayerDidFinishPlaying(_:successfully:)`      | После завершения воспроизведения (duration достигнут или файл закончился)       | Перейти к следующему треку, обновить UI (play → pause), зациклить     | Самый важный метод — обработка конца трека |
| `audioPlayerDecodeErrorDidOccur(_:error:)`          | При ошибке декодирования файла (повреждённый mp3, неподдерживаемый формат)      | Показать алерт, пропустить трек, логировать ошибку                   | Обработка битых файлов |
| `audioPlayerBeginInterruption(_:)`                  | Когда аудиосессия прервана (звонок, Siri, другое приложение)                    | Поставить на паузу, сохранить currentTime                             | Устаревший (iOS 8+ используйте AVAudioSession interruption) |
| `audioPlayerEndInterruption(_:withOptions:)`        | После окончания прерывания                                                      | Возобновить воспроизведение, если `.shouldResume`                     | Устаревший, но иногда нужен |
| `audioPlayer(_:didSeekToTime:)` (редко)             | При перемотке (seek) — не всегда вызывается                                     | Обновление UI прогресса                                               | Редко используется |

### Самый популярный и рекомендуемый паттерн 2026 года  
([[AVAudioPlayer]] + AVAudioPlayerDelegate + [[AVAudioSession]] + прогресс + обработка конца)

```swift
import UIKit
import AVFoundation

class MusicPlayerViewController: UIViewController, AVAudioPlayerDelegate {
    
    private var player: AVAudioPlayer?
    private var displayLink: CADisplayLink?
    
    private lazy var playPauseButton: UIButton = {
        let btn = UIButton(type: .system)
        btn.setTitle("Play", for: .normal)
        btn.addTarget(self, action: #selector(togglePlayback), for: .touchUpInside)
        return btn
    }()
    
    private lazy var progressSlider: UISlider = {
        let s = UISlider()
        s.minimumValue = 0
        s.maximumValue = 1
        s.addTarget(self, action: #selector(seek), for: .valueChanged)
        return s
    }()
    
    private lazy var timeLabel: UILabel = {
        let lbl = UILabel()
        lbl.text = "00:00 / 00:00"
        lbl.textAlignment = .center
        return lbl
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        
        setupAudioSession()
        loadAudio()
        setupUI()
    }
    
    private func setupAudioSession() {
        do {
            let session = AVAudioSession.sharedInstance()
            try session.setCategory(.playback, mode: .default)
            try session.setActive(true)
        } catch {
            print("Ошибка AVAudioSession: \(error)")
        }
    }
    
    private func loadAudio() {
        guard let url = Bundle.main.url(forResource: "song", withExtension: "mp3") else { return }
        
        do {
            player = try AVAudioPlayer(contentsOf: url)
            player?.delegate = self
            player?.prepareToPlay()
            player?.numberOfLoops = 0  // 0 = один раз, -1 = бесконечно
            updateTimeLabel()
        } catch {
            print("Ошибка загрузки: \(error)")
        }
    }
    
    @objc private func togglePlayback() {
        guard let player else { return }
        
        if player.isPlaying {
            player.pause()
            playPauseButton.setTitle("Play", for: .normal)
            stopProgress()
        } else {
            player.play()
            playPauseButton.setTitle("Pause", for: .normal)
            startProgress()
        }
    }
    
    private func startProgress() {
        displayLink = CADisplayLink(target: self, selector: #selector(updateProgress))
        displayLink?.add(to: .main, forMode: .default)
    }
    
    private func stopProgress() {
        displayLink?.invalidate()
        displayLink = nil
    }
    
    @objc private func updateProgress() {
        guard let player else { return }
        
        let progress = Float(player.currentTime / player.duration)
        progressSlider.value = progress
        
        let current = Int(player.currentTime)
        let total = Int(player.duration)
        timeLabel.text = "\(formatTime(current)) / \(formatTime(total))"
    }
    
    private func formatTime(_ seconds: Int) -> String {
        String(format: "%02d:%02d", seconds / 60, seconds % 60)
    }
    
    @objc private func seek(_ sender: UISlider) {
        guard let player else { return }
        player.currentTime = TimeInterval(sender.value * Float(player.duration))
        updateProgress()
    }
    
    // MARK: - AVAudioPlayerDelegate
    
    func audioPlayerDidFinishPlaying(_ player: AVAudioPlayer, successfully flag: Bool) {
        playPauseButton.setTitle("Play", for: .normal)
        progressSlider.value = 0
        updateTimeLabel()
        stopProgress()
        
        if flag {
            print("Трек успешно завершён")
        } else {
            print("Завершён с ошибкой")
        }
    }
    
    func audioPlayerDecodeErrorDidOccur(_ player: AVAudioPlayer, error: Error?) {
        print("Ошибка декодирования: \(error?.localizedDescription ?? "неизвестно")")
        playPauseButton.setTitle("Play", for: .normal)
    }
    
    func audioPlayerBeginInterruption(_ player: AVAudioPlayer) {
        // Устаревший метод (iOS 8+ используйте AVAudioSession interruption)
        print("Прерывание начато")
        player.pause()
    }
    
    func audioPlayerEndInterruption(_ player: AVAudioPlayer, withOptions flags: Int) {
        // Устаревший
        if flags & AVAudioSession.InterruptionOptions.shouldResume.rawValue != 0 {
            player.play()
        }
    }
}
```

### Лучшие практики AVAudioPlayerDelegate в 2026 году

- **Обязательно** реализуйте `audioPlayerDidFinishPlaying(_:successfully:)`  
- **Для прерываний** — в 2026 году используйте `AVAudioSession.interruptionNotification` вместо устаревших методов делегата  
- **Для прогресса** — используйте [[CADisplayLink]] или [[Timer]] (0.1–0.5 сек) + `currentTime / duration`  
- **prepareToPlay()** — вызывайте перед первым `play()`  
- **Фон** — настройте `.playback` категорию + `UIBackgroundModes` → `audio` в Info.plist  
- **Для нескольких треков** — используйте `AVQueuePlayer` или `AVAudioEngine`  
- **Для [[SwiftUI]]** — оборачивайте в `UIViewControllerRepresentable`  
- **Документируйте** — пишите комментарий:

```swift
extension MusicPlayerViewController: AVAudioPlayerDelegate {
    func audioPlayerDidFinishPlaying(_ player: AVAudioPlayer, successfully flag: Bool) {
        // Переход к следующему треку или остановка
    }
}
```

**Короткий итог 2026**:
> **AVAudioPlayerDelegate** — протокол для обработки **событий** воспроизведения в AVAudioPlayer.  
> В 2026 году:  
> - ключевой метод — `audioPlayerDidFinishPlaying(_:successfully:)`  
> - прерывания лучше обрабатывать через `AVAudioSession` notification  
> - самый популярный паттерн — delegate + прогресс через CADisplayLink + обработка конца трека  
> - идеален для локального аудио (музыка, подкасты, эффекты)  
> - в SwiftUI — используйте обёртку или `AVPlayer`  
