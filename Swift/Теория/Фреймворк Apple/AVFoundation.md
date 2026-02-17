**AVFoundation** — это фундаментальный фреймворк Apple для работы с **аудио**, **видео**, **захватом медиа** и **обработкой медиаданных** на всех платформах Apple ([[iOS]], iPadOS, macOS, watchOS, tvOS, visionOS).

По состоянию на февраль 2026 года AVFoundation остаётся **основным и самым мощным** низкоуровневым инструментом для любых задач, связанных с медиа, особенно там, где требуется полный контроль или высокая производительность.

### Основные подфреймворки и классы AVFoundation (актуальные 2026)

| Подзадача / Функциональность            | Главные классы / протоколы                                          | Когда использовать в 2026 году                          | Современная альтернатива / примечание                     |
| --------------------------------------- | ------------------------------------------------------------------- | ------------------------------------------------------- | --------------------------------------------------------- |
| Захват видео и аудио с камеры/микрофона | `AVCaptureSession`, `AVCaptureDevice`, `AVCaptureVideoDataOutput`   | Реал-тайм обработка кадров, кастомные камеры, AR/Vision | RealityKit / ARKit для AR, AVFoundation для raw-кадров    |
| Запись видео/аудио                      | `AVAssetWriter`, `AVAssetWriterInput`, `AVCaptureMovieFileOutput`   | Кастомная запись, обработка на лету                     | `AVAssetExportSession` для простых случаев                |
| Воспроизведение аудио/видео             | `AVPlayer`, `AVPlayerItem`, `AVPlayerLayer`, `AVAudioPlayer`        | Плееры, стриминг, background playback                   | AVPlayer → основной, AVAudioEngine для продвинутого аудио |
| Обработка аудио (эффекты, микширование) | `AVAudioEngine`, `AVAudioPlayerNode`, `AVAudioUnit`                 | Эквалайзер, реверб, запись/микширование в реал-тайм     | AVAudioEngine — современный стандарт                      |
| Работа с медиафайлами и метаданными     | `AVAsset`, `AVAssetTrack`, `AVMetadataItem`, `AVMutableComposition` | Редактирование, извлечение метаданных, композиция       | AVAsset + AVComposition — мощно                           |
| Распознавание речи / транскрипция       | `SFSpeechRecognizer` (Speech фреймворк) + AVAudioEngine             | Живое распознавание, субтитры                           | Speech + AVFoundation                                     |
| Vision + AVFoundation                   | `AVCaptureVideoDataOutput` + Vision requests                        | Реал-тайм детекция объектов, лиц, текста                | Vision + AVCapture                                        |
| Live Activities + Push + AVFoundation   | `AVPlayer` + Push Notifications                                     | Фоновое воспроизведение + уведомления о смене трека     | [[iOS]] 18+ интеграция                                    |

### Самые популярные и рекомендуемые паттерны AVFoundation в 2026 году

#### 1. Простой захват видео + обработка кадров в реальном времени (самый частый сценарий)

```swift
import AVFoundation

class CameraManager: NSObject, AVCaptureVideoDataOutputSampleBufferDelegate {
    
    private let session = AVCaptureSession()
    private let videoOutput = AVCaptureVideoDataOutput()
    private var previewLayer: AVCaptureVideoPreviewLayer?
    
    func setup() async throws {
        // 1. Запрос разрешения
        guard await AVCaptureDevice.requestAccess(for: .video) else {
            throw CameraError.permissionDenied
        }
        
        // 2. Настройка сессии
        session.beginConfiguration()
        session.sessionPreset = .high
        
        // 3. Устройство ввода
        guard let device = AVCaptureDevice.default(for: .video),
              let input = try? AVCaptureDeviceInput(device: device) else {
            throw CameraError.noDevice
        }
        
        if session.canAddInput(input) {
            session.addInput(input)
        }
        
        // 4. Вывод видео-данных
        videoOutput.setSampleBufferDelegate(self, queue: .main)
        if session.canAddOutput(videoOutput) {
            session.addOutput(videoOutput)
        }
        
        session.commitConfiguration()
    }
    
    func start() {
        previewLayer = AVCaptureVideoPreviewLayer(session: session)
        previewLayer?.videoGravity = .resizeAspectFill
        session.startRunning()
    }
    
    func stop() {
        session.stopRunning()
    }
    
    // Делегат — обработка каждого кадра
    func captureOutput(_ output: AVCaptureOutput, didOutput sampleBuffer: CMSampleBuffer, from connection: AVCaptureConnection) {
        // Здесь можно: Vision, Core Image, Metal, отправка на сервер и т.д.
        guard let pixelBuffer = CMSampleBufferGetImageBuffer(sampleBuffer) else { return }
        
        // Пример: простой фильтр Core Image
        let ciImage = CIImage(cvPixelBuffer: pixelBuffer)
        // обработка ciImage...
    }
}
```

#### 2. Простой плеер с AVPlayer (самый частый для видео)

```swift
import AVFoundation
import AVKit

class VideoPlayerManager {
    private var player: AVPlayer?
    private var playerLayer: AVPlayerLayer?
    
    func play(url: URL, in view: UIView) {
        let item = AVPlayerItem(url: url)
        player = AVPlayer(playerItem: item)
        
        playerLayer = AVPlayerLayer(player: player)
        playerLayer?.frame = view.bounds
        playerLayer?.videoGravity = .resizeAspect
        view.layer.addSublayer(playerLayer!)
        
        player?.play()
        
        // Обработка состояний
        NotificationCenter.default.addObserver(
            forName: .AVPlayerItemDidPlayToEndTime,
            object: item,
            queue: .main
        ) { _ in
            self.player?.seek(to: .zero)
            self.player?.play()
        }
    }
    
    func pause() {
        player?.pause()
    }
}
```

### Лучшие практики AVFoundation в Swift 2026

- **Всегда запрашивай разрешения** асинхронно (`requestAccess(for:)`)  
- **Используй actor** для хранения состояния (камера, плеер, буферы)  
- **AVCaptureSession** — запускай/останавливай в [[viewWillAppear]] / [[viewWillDisappear]]  
- **AVPlayer** — используй `AVPlayerViewController` для простых плееров, `AVPlayerLayer` для кастомных  
- **AVAudioEngine** — современный способ работы с аудио (заменил старый AVAudioPlayer в большинстве случаев)  
- **Vision + AVCapture** — стандарт для реал-тайм компьютерного зрения  
- **Swift 6 strict concurrency** — весь UI и захват — `@MainActor`, обработка кадров — `Task.detached` или dedicated actor  
- **Тестирование** — используй `AVAssetWriter` mocks или `AVCaptureDeviceInput` stubs  
- **Документируйте** — пишите комментарий «AVFoundation — захват видео + Vision processing»

**Короткий девиз 2026**:
> «AVFoundation — это когда тебе нужен полный контроль над аудио/видео на уровне устройства.  
> В 2026 году это **лучший выбор** для кастомных камер, плееров, обработки в реал-тайм и интеграции с Vision / Metal.  
> Для простых плееров и аудио — используй AVPlayer / AVAudioPlayer, для сложного — AVFoundation + actor.»
