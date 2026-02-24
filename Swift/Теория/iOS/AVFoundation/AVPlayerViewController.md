**AVPlayerViewController** — это готовый системный контроллер Apple для воспроизведения видео и аудио в [[iOS]]-приложениях.

Он предоставляет **полноценный плеер** с нативным интерфейсом (как в Apple TV, YouTube, Netflix, встроенном видео в Safari и т.д.), включая:

- элементы управления (play/pause, слайдер времени, полноэкранный режим)
- поддержку AirPlay, Picture-in-Picture (PiP)
- субтитры, выбор аудиодорожки
- жесты (тап → показать/скрыть контролы, двойной тап → перемотка)
- автоматическую адаптацию под ориентацию и размер экрана

Это **самый рекомендуемый** и **самый часто используемый** способ воспроизведения видео в UIKit-приложениях в 2025–2026 годах.

### Основные сценарии использования (2026 актуально)

| Сценарий                                    | Рекомендуемый подход                    | Когда использовать AVPlayerViewController |
| ------------------------------------------- | --------------------------------------- | ----------------------------------------- |
| Простое воспроизведение одного видео        | `AVPlayerViewController` + [[AVPlayer]] | Почти всегда — самый простой и надёжный   |
| Видео из URL (локальное или удалённое)      | Да                                      | Основной кейс                             |
| Видео из AVAsset / AVPlayerItem             | Да                                      | Когда нужна кастомная подготовка актива   |
| Встроенное видео в интерфейсе (не модально) | `AVPlayerViewController` как child VC   | Экран с видео + текст/кнопки вокруг       |
| Полноэкранный плеер                         | `present` или `push`                    | Стандартный полноэкранный просмотр        |
| Picture-in-Picture (PiP)                    | Автоматически поддерживается            | Включить `allowsPictureInPicturePlayback` |
| AirPlay                                     | Автоматически поддерживается            | `allowsExternalPlayback = true`           |

### Самый популярный и рекомендуемый паттерн (2026)

```swift
import UIKit
import AVKit
import AVFoundation

class VideoPlayerViewController: UIViewController {
    
    private var playerViewController: AVPlayerViewController!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .black
        
        setupPlayerViewController()
        playVideo(from: "https://example.com/video.mp4")
    }
    
    private func setupPlayerViewController() {
        playerViewController = AVPlayerViewController()
        
        // Самые важные настройки
        playerViewController.showsPlaybackControls = true          // показывать контролы
        playerViewController.updatesNowPlayingInfoCenter = true    // интеграция с Control Center / Lock Screen
        playerViewController.allowsPictureInPicturePlayback = true // PiP
        playerViewController.canStartPictureInPictureAutomaticallyFromInline = true
        
        // Добавляем как child-контроллер
        addChild(playerViewController)
        view.addSubview(playerViewController.view)
        playerViewController.view.frame = view.bounds
        playerViewController.didMove(toParent: self)
        
        // Авто-респект safe area и повороты
        playerViewController.view.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            playerViewController.view.topAnchor.constraint(equalTo: view.topAnchor),
            playerViewController.view.bottomAnchor.constraint(equalTo: view.bottomAnchor),
            playerViewController.view.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            playerViewController.view.trailingAnchor.constraint(equalTo: view.trailingAnchor)
        ])
    }
    
    private func playVideo(from urlString: String) {
        guard let url = URL(string: urlString) else { return }
        
        let player = AVPlayer(url: url)
        playerViewController.player = player
        
        // Опционально: автоматический старт
        player.playImmediately(atRate: 1.0)
        
        // Подписка на события (опционально)
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(playerDidFinishPlaying),
            name: .AVPlayerItemDidPlayToEndTime,
            object: player.currentItem
        )
    }
    
    @objc private func playerDidFinishPlaying() {
        // зацикливание или показ следующего видео
        playerViewController.player?.seek(to: .zero)
        playerViewController.player?.play()
    }
    
    deinit {
        NotificationCenter.default.removeObserver(self)
    }
}
```

### Полный пример с локальным файлом и модальным показом

```swift
@objc private func playLocalVideo() {
    guard let url = Bundle.main.url(forResource: "intro", withExtension: "mp4") else { return }
    
    let player = AVPlayer(url: url)
    let playerVC = AVPlayerViewController()
    playerVC.player = player
    playerVC.showsPlaybackControls = true
    
    present(playerVC, animated: true) {
        player.play()
    }
}
```

### Лучшие практики AVPlayerViewController в 2026 году

- **Всегда** добавляйте как **child view controller** если видео — часть экрана (не модально)  
- **Включайте** `showsPlaybackControls = true` — пользователи ожидают стандартный плеер Apple  
- **Для PiP** — обязательно `allowsPictureInPicturePlayback = true` + `canStartPictureInPictureAutomaticallyFromInline = true`  
- **Для фонового воспроизведения** — настройте `AVAudioSession` в `AppDelegate` / `SceneDelegate`  
- **Для AirPlay** — `allowsExternalPlayback = true` (по умолчанию true)  
- **Для субтитров** — добавляйте `AVPlayerItem` с внешними субтитрами через `AVMutableComposition`  
- **Для SwiftUI** — используйте `VideoPlayer(player: AVPlayer)` — AVPlayerViewController нужен только в UIKit или смешанных проектах  
- **Для обработки ошибок** — подпишитесь на `AVPlayerItem.failed` и `AVPlayerItem.newErrorLogEntryNotification`  
- **Документируйте** — пишите комментарий «AVPlayerViewController — встроенный полноэкранный плеер с поддержкой PiP и AirPlay»

**Короткий итог 2026**:
> AVPlayerViewController — это **готовый системный видеоплеер** Apple с нативным интерфейсом.  
> В 2026 году:  
> - основной способ — `AVPlayerViewController` + `AVPlayer(url:)`  
> - ключевые настройки — `showsPlaybackControls`, `allowsPictureInPicturePlayback`, `updatesNowPlayingInfoCenter`  
> - идеально для: встроенного видео, полноэкранного просмотра, PiP, AirPlay  
> - в [[SwiftUI]] — используйте `VideoPlayer`, в [[UIKit]] — это **единственный рекомендуемый** нативный плеер  
