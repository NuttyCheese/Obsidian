**AVPlayer** — это центральный класс из **AVFoundation** для воспроизведения аудио и видео в приложениях Apple (iOS, iPadOS, macOS, tvOS, visionOS).  
Он поддерживает практически все современные форматы (H.264, HEVC/H.265, AV1, MP4, HLS, DASH, FairPlay DRM и т.д.) и является **единственным рекомендуемым** инструментом для нативного воспроизведения медиа в 2026 году.

### Основные возможности AVPlayer (актуальные на 2026 год)

| Возможность                              | Ключевые классы / методы                        | Когда использовать в 2026 году                          | Примечание / ограничение |
|------------------------------------------|--------------------------------------------------|----------------------------------------------------------|---------------------------|
| Воспроизведение локальных файлов         | `AVPlayer(url:)` / `AVPlayerItem(url:)`         | Видео/аудио из Bundle / Documents                        | Поддерживает все форматы iOS |
| HLS / DASH стриминг                      | `AVPlayerItem(asset: AVURLAsset)`               | Live / VOD стримы (YouTube, Twitch, Apple Music)         | Лучшая поддержка адаптивного битрейта |
| FairPlay DRM (защищённый контент)        | `AVPlayerItem` + `AVAssetResourceLoaderDelegate` | Netflix, Apple TV+, Disney+, HBO Max                     | Требует сертификат и сервер SKD |
| AirPlay 2 / внешние устройства           | `allowsExternalPlayback = true`                 | Телевизоры, колонки, CarPlay                             | Автоматически работает |
| Picture-in-Picture (PiP)                 | `allowsPictureInPicturePlayback = true`         | iOS 14+ (включая multitasking)                           | Требует background modes |
| Фоновое воспроизведение                  | `AVAudioSession.Category.playback`              | Музыка / подкасты при заблокированном экране             | Обязательно настроить AVAudioSession |
| Временные метки / chapters               | `currentTime()`, `seek(to:)`                    | Пропуск рекламы, главы в видео                           | Точное управление |
| Метаданные (title, artwork, chapters)    | `AVMetadataItem`, `nowPlayingInfoCenter`        | Отображение в Lock Screen / Control Center               | MPNowPlayingInfoCenter |
| AirPlay маршрутизация                    | `AVRoutePickerView`, `AVPlayer.airPlayVideoRouteActive` | Выбор колонки / телевизора                               | iOS 11+ |
| VideoToolbox / Hardware acceleration     | Автоматически (H.264/HEVC/AV1)                  | Энергоэффективное воспроизведение                        | A17 Pro / M4 и выше — AV1 |

### Самый популярный и рекомендуемый паттерн 2026 года

```swift
import AVKit
import AVFoundation

@MainActor
class VideoPlayerViewController: UIViewController {
    
    private var player: AVPlayer?
    private var playerLayer: AVPlayerLayer?
    private var playerItem: AVPlayerItem?
    
    private let videoURL: URL
    
    init(url: URL) {
        self.videoURL = url
        super.init(nibName: nil, bundle: nil)
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        setupAudioSession()
        setupPlayer()
        setupControls()
    }
    
    private func setupAudioSession() {
        do {
            let session = AVAudioSession.sharedInstance()
            try session.setCategory(.playback, mode: .moviePlayback)
            try session.setActive(true)
        } catch {
            print("Ошибка аудио-сессии: \(error)")
        }
    }
    
    private func setupPlayer() {
        let asset = AVURLAsset(url: videoURL)
        playerItem = AVPlayerItem(asset: asset)
        
        player = AVPlayer(playerItem: playerItem)
        player?.automaticallyWaitsToMinimizeStalling = true
        
        playerLayer = AVPlayerLayer(player: player)
        playerLayer?.videoGravity = .resizeAspect
        playerLayer?.frame = view.bounds
        view.layer.addSublayer(playerLayer!)
        
        // Наблюдение за состоянием
        playerItem?.addObserver(self, forKeyPath: #keyPath(AVPlayerItem.status), options: [.new], context: nil)
    }
    
    override func observeValue(forKeyPath keyPath: String?, of object: Any?, change: [NSKeyValueChangeKey : Any]?, context: UnsafeMutableRawPointer?) {
        if keyPath == #keyPath(AVPlayerItem.status),
           let item = object as? AVPlayerItem {
            
            switch item.status {
            case .readyToPlay:
                player?.play()
            case .failed:
                print("Ошибка воспроизведения: \(item.error?.localizedDescription ?? "неизвестно")")
            default:
                break
            }
        }
    }
    
    private func setupControls() {
        // Добавляем AVPlayerViewController или кастомные кнопки
        let playerVC = AVPlayerViewController()
        playerVC.player = player
        playerVC.showsPlaybackControls = true
        
        addChild(playerVC)
        view.addSubview(playerVC.view)
        playerVC.view.frame = view.bounds
        playerVC.didMove(toParent: self)
    }
    
    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
        playerLayer?.frame = view.bounds
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        player?.pause()
    }
    
    deinit {
        playerItem?.removeObserver(self, forKeyPath: #keyPath(AVPlayerItem.status))
    }
}
```

### Лучшие практики AVPlayer в Swift 2026

- **AVPlayerViewController** — для простого воспроизведения (встроенные контролы, PiP, AirPlay)  
- **AVPlayerLayer** — для кастомного UI (оверлеи, субтитры, кастомные кнопки)  
- **AVAudioSession** — обязательно настрой `.playback` категорию  
- **@MainActor** — весь UI-контроллер и обновления — на главном акторе  
- **KVO** — наблюдай за `status`, `timeControlStatus`, `reasonForWaitingToPlay`  
- **Swift 6 strict concurrency** — AVPlayer thread-safe, но UI-обновления — только @MainActor  
- **Фоновое воспроизведение** — настрой `AVAudioSession` + background modes  
- **Прогресс / время** — используй `periodicTimeObserver`  
- **Документируйте** — пиши комментарий «AVPlayer — воспроизведение HLS / локального видео с контролами»

**Короткий девиз 2026**:
> «AVPlayer — это когда тебе нужно воспроизводить видео/аудио с поддержкой HLS, FairPlay, PiP, AirPlay и фонового режима.  
> В 2026 году это **единственный** рекомендуемый инструмент Apple для нативного медиа-плеера.  
> Для простых случаев — AVPlayerViewController, для кастомного UI — AVPlayer + AVPlayerLayer.»

Удачи с плавным и энергоэффективным воспроизведением медиа в Swift! ▶️