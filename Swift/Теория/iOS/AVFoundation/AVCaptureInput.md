#avfoundation #capture #input #camera #microphone #avcaptureinput #avcapturedeviceinput`

---
### Определение
**AVCaptureInput** — это абстрактный базовый класс во фреймворке [[AVFoundation]], который представляет собой источник данных для сессии захвата ([[AVCaptureSession]]). Он является точкой входа, через которую медиаданные (видео, аудио, метаданные) поступают в систему для дальнейшей обработки и вывода .

Простыми словами, `AVCaptureInput` — это "штекер", который подключает физическое устройство (камеру, микрофон) к виртуальному конвейеру обработки ([[AVCaptureSession]]). Без входов сессия не имеет данных для работы.

### Иерархия классов

`AVCaptureInput` имеет несколько конкретных подклассов:

1.  **[[AVCaptureDeviceInput]]** — самый распространенный тип. Создается из [[AVCaptureDevice]] (камера, микрофон).
2.  **[[AVCapturePort]]** — представляет конкретный порт входа (например, отдельный видеопоток от камеры).
3.  **[[AVCaptureScreenInput]]** — для захвата экрана (macOS).
4.  **[[AVCaptureFileInput]]** — для чтения медиаданных из файла.
5.  **[[AVCaptureMetadataInput]]** — для ручного ввода метаданных (специализированный).

В iOS-разработке мы почти всегда работаем с **AVCaptureDeviceInput**.

### Зачем это знать iOS-разработчику?
1.  **Подключение устройств:** Добавление камеры и микрофона в сессию захвата.
2.  **Управление устройствами:** Переключение между камерами (фронтальная/задняя).
3.  **Конфигурация сессии:** Добавление и удаление входов во время работы сессии.
4.  **Получение портов:** Доступ к конкретным потокам данных (например, отдельно к видео и аудио от одного устройства).
5.  **Проверка совместимости:** Убедиться, что устройство может быть добавлено в сессию.

---

### Архитектура и место в AVCaptureSession

```mermaid
graph TD
    subgraph Devices [Физические устройства]
        A[AVCaptureDevice<br>Камера]
        B[AVCaptureDevice<br>Микрофон]
    end

    subgraph Inputs [AVCaptureInput]
        C[AVCaptureDeviceInput<br>(из камеры)]
        D[AVCaptureDeviceInput<br>(из микрофона)]
    end

    subgraph Session [AVCaptureSession]
        E[AVCaptureSession<br>Конвейер]
    end

    subgraph Outputs [AVCaptureOutput]
        F[AVCaptureMovieFileOutput]
        G[AVCaptureVideoDataOutput]
        H[AVCaptureAudioDataOutput]
    end

    A -->|AVCaptureDeviceInput| C
    B -->|AVCaptureDeviceInput| D
    
    C -->|Добавляется в| E
    D -->|Добавляется в| E
    
    E -->|Передает данные| F
    E -->|Передает данные| G
    E -->|Передает данные| H
    
    style Inputs fill:#ccf,stroke:#333,stroke-width:2px
    style C fill:#cfc,stroke:#333
    style D fill:#cfc,stroke:#333
    style Session fill:#f9f,stroke:#333
```

### Ключевые свойства и методы

#### AVCaptureInput (базовый класс)
- `ports` — массив объектов `AVCaptureInput.Port`, представляющих отдельные потоки данных от этого входа.
- `- connection(with:)` — получить соединение для конкретного типа медиа (реже используется, обычно получают от выходов).

#### AVCaptureDeviceInput (основной рабочий класс)
- `init(device:)` — создает вход из устройства.
- `device` — устройство, из которого создан вход (только для чтения).
- `- ports` — унаследованное свойство, содержит порты для видео и аудио.

#### Создание входа
```swift
guard let camera = AVCaptureDevice.default(.builtInWideAngleCamera, for: .video, position: .back),
      let input = try? AVCaptureDeviceInput(device: camera) else {
    return
}
```

---

### Примеры от простого к сложному

#### Уровень 0: Базовая структура с добавлением входов

```swift
import UIKit
import AVFoundation

class InputDemoViewController: UIViewController {
    
    var captureSession: AVCaptureSession!
    var videoInput: AVCaptureDeviceInput?
    var audioInput: AVCaptureDeviceInput?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupSession()
    }
    
    private func setupSession() {
        captureSession = AVCaptureSession()
        
        // Добавляем входы
        addVideoInput()
        addAudioInput()
        
        // Если есть хотя бы один вход, запускаем сессию
        if captureSession.inputs.isEmpty {
            print("Нет доступных входов")
            return
        }
        
        // Здесь можно добавить выходы...
        
        DispatchQueue.global(qos: .userInitiated).async { [weak self] in
            self?.captureSession.startRunning()
        }
    }
    
    private func addVideoInput() {
        guard let camera = AVCaptureDevice.default(.builtInWideAngleCamera, for: .video, position: .back) else {
            print("Камера не найдена")
            return
        }
        
        do {
            videoInput = try AVCaptureDeviceInput(device: camera)
            
            if captureSession.canAddInput(videoInput!) {
                captureSession.addInput(videoInput!)
                print("Видео вход добавлен")
            }
        } catch {
            print("Ошибка создания видео входа: \(error.localizedDescription)")
        }
    }
    
    private func addAudioInput() {
        guard let microphone = AVCaptureDevice.default(for: .audio) else {
            print("Микрофон не найден")
            return
        }
        
        do {
            audioInput = try AVCaptureDeviceInput(device: microphone)
            
            if captureSession.canAddInput(audioInput!) {
                captureSession.addInput(audioInput!)
                print("Аудио вход добавлен")
            }
        } catch {
            print("Ошибка создания аудио входа: \(error.localizedDescription)")
        }
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        DispatchQueue.global(qos: .background).async { [weak self] in
            self?.captureSession.stopRunning()
        }
    }
}
```

#### Уровень 1: Работа с портами входа
Каждый вход может иметь несколько портов (например, видео и аудио от одной камеры со встроенным микрофоном).

```swift
import UIKit
import AVFoundation

class PortsViewController: InputDemoViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // После настройки сессии исследуем порты
        DispatchQueue.main.asyncAfter(deadline: .now() + 1) { [weak self] in
            self?.inspectInputs()
        }
    }
    
    func inspectInputs() {
        print("=== ИНФОРМАЦИЯ О ВХОДАХ ===")
        
        for (index, input) in captureSession.inputs.enumerated() {
            print("Вход #\(index + 1): \(input)")
            
            for (portIndex, port) in input.ports.enumerated() {
                print("  Порт #\(portIndex + 1):")
                print("    - Медиа тип: \(port.mediaType.rawValue)")
                print("    - Формат: \(port.formatDescription?.mediaSubType.description ?? "unknown")")
                print("    - Включен: \(port.isEnabled)")
                
                if port.mediaType == .video {
                    print("    - Это видео порт")
                } else if port.mediaType == .audio {
                    print("    - Это аудио порт")
                }
            }
        }
        
        // Можно получить конкретный порт по типу
        if let videoInput = videoInput {
            let videoPorts = videoInput.ports.filter { $0.mediaType == .video }
            print("Видео портов: \(videoPorts.count)")
            
            let audioPorts = videoInput.ports.filter { $0.mediaType == .audio }
            print("Аудио портов (от камеры): \(audioPorts.count)")
        }
    }
}
```

#### Уровень 2: Переключение между камерами (замена входа)
Демонстрация того, как удалить старый вход и добавить новый.

```swift
import UIKit
import AVFoundation

class CameraSwitchViewController: InputDemoViewController {
    
    let switchButton = UIButton()
    var currentCameraPosition: AVCaptureDevice.Position = .back
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupSwitchButton()
    }
    
    private func setupSwitchButton() {
        switchButton.setTitle("🔄 Переключить камеру", for: .normal)
        switchButton.backgroundColor = .blue
        switchButton.layer.cornerRadius = 10
        switchButton.frame = CGRect(x: view.bounds.midX - 100, 
                                    y: view.bounds.height - 200, 
                                    width: 200, 
                                    height: 50)
        switchButton.addTarget(self, action: #selector(switchCamera), for: .touchUpInside)
        view.addSubview(switchButton)
    }
    
    @objc func switchCamera() {
        // Определяем новую позицию
        let newPosition: AVCaptureDevice.Position = (currentCameraPosition == .back) ? .front : .back
        
        // Ищем устройство с новой позицией
        guard let newCamera = AVCaptureDevice.default(.builtInWideAngleCamera, for: .video, position: newPosition),
              let newInput = try? AVCaptureDeviceInput(device: newCamera) else {
            print("Не удалось создать вход для новой камеры")
            return
        }
        
        // Конфигурируем сессию
        captureSession.beginConfiguration()
        
        // Удаляем старый видео вход
        if let oldVideoInput = videoInput {
            captureSession.removeInput(oldVideoInput)
        }
        
        // Добавляем новый, если возможно
        if captureSession.canAddInput(newInput) {
            captureSession.addInput(newInput)
            videoInput = newInput
            currentCameraPosition = newPosition
            print("Камера переключена на: \(newPosition == .back ? "заднюю" : "фронтальную")")
        } else {
            // Если не получилось, возвращаем старый вход
            if let oldVideoInput = videoInput {
                captureSession.addInput(oldVideoInput)
            }
        }
        
        captureSession.commitConfiguration()
    }
}
```

#### Уровень 3: Временное отключение входа
Можно отключить вход, не удаляя его из сессии.

```swift
import UIKit
import AVFoundation

class DisableInputViewController: InputDemoViewController {
    
    let toggleVideoButton = UIButton()
    let toggleAudioButton = UIButton()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupToggleButtons()
    }
    
    private func setupToggleButtons() {
        toggleVideoButton.setTitle("Выключить видео", for: .normal)
        toggleVideoButton.backgroundColor = .blue
        toggleVideoButton.frame = CGRect(x: 20, y: 120, width: 150, height: 40)
        toggleVideoButton.addTarget(self, action: #selector(toggleVideo), for: .touchUpInside)
        view.addSubview(toggleVideoButton)
        
        toggleAudioButton.setTitle("Выключить аудио", for: .normal)
        toggleAudioButton.backgroundColor = .blue
        toggleAudioButton.frame = CGRect(x: view.bounds.width - 170, y: 120, width: 150, height: 40)
        toggleAudioButton.addTarget(self, action: #selector(toggleAudio), for: .touchUpInside)
        view.addSubview(toggleAudioButton)
    }
    
    @objc func toggleVideo() {
        guard let videoInput = videoInput else { return }
        
        // Отключаем все видео-порты входа
        let videoPorts = videoInput.ports.filter { $0.mediaType == .video }
        
        captureSession.beginConfiguration()
        
        for port in videoPorts {
            port.isEnabled.toggle()
        }
        
        captureSession.commitConfiguration()
        
        let status = videoPorts.first?.isEnabled == true ? "Включено" : "Выключено"
        toggleVideoButton.setTitle("\(status) видео", for: .normal)
        toggleVideoButton.backgroundColor = videoPorts.first?.isEnabled == true ? .blue : .gray
    }
    
    @objc func toggleAudio() {
        // Проверяем аудио-порты на видео входе (если камера имеет микрофон)
        if let videoInput = videoInput {
            let audioPorts = videoInput.ports.filter { $0.mediaType == .audio }
            
            captureSession.beginConfiguration()
            for port in audioPorts {
                port.isEnabled.toggle()
            }
            captureSession.commitConfiguration()
            
            let status = audioPorts.first?.isEnabled == true ? "Включено" : "Выключено"
            toggleAudioButton.setTitle("\(status) аудио (камера)", for: .normal)
        }
        
        // Также можно отключить отдельный аудио вход
        if let audioInput = audioInput {
            // Для отдельного аудио входа нужно отключать его целиком, а не порты
            // Или использовать connections из выходов
        }
    }
}
```

#### Уровень 4: Проверка поддержки форматов и настройка
Перед созданием входа можно проверить, поддерживает ли устройство нужные параметры.

```swift
import UIKit
import AVFoundation

class AdvancedInputViewController: InputDemoViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        checkCameraCapabilities()
    }
    
    func checkCameraCapabilities() {
        // Получаем все камеры
        let discoverySession = AVCaptureDevice.DiscoverySession(deviceTypes: [.builtInWideAngleCamera, .builtInTelephotoCamera, .builtInUltraWideCamera], 
                                                                 mediaType: .video, 
                                                                 position: .unspecified)
        
        let cameras = discoverySession.devices
        print("Найдено камер: \(cameras.count)")
        
        for camera in cameras {
            print("\n=== Камера: \(camera.localizedName) ===")
            print("Позиция: \(camera.position == .back ? "задняя" : (camera.position == .front ? "фронтальная" : "другая"))")
            
            // Проверяем поддержку форматов
            let formats = camera.formats
            print("Поддерживает \(formats.count) форматов:")
            
            // Показываем первые 3 формата
            for (index, format) in formats.prefix(3).enumerated() {
                let dimensions = CMVideoFormatDescriptionGetDimensions(format.formatDescription)
                let fps = format.videoSupportedFrameRateRanges.first?.maxFrameRate ?? 0
                print("  Формат \(index + 1): \(dimensions.width)x\(dimensions.height) @ \(Int(fps)) fps")
            }
            
            // Проверяем, можно ли создать вход
            do {
                let input = try AVCaptureDeviceInput(device: camera)
                print("  ✅ Можно создать вход")
            } catch {
                print("  ❌ Нельзя создать вход: \(error.localizedDescription)")
            }
        }
    }
    
    // Создание входа с определенным форматом
    func createInputWithPreferredFormat() -> AVCaptureDeviceInput? {
        guard let camera = AVCaptureDevice.default(.builtInWideAngleCamera, for: .video, position: .back) else {
            return nil
        }
        
        do {
            // Пытаемся настроить устройство на предпочтительный формат
            try camera.lockForConfiguration()
            
            // Ищем формат 1920x1080 @ 60 fps
            if let preferredFormat = camera.formats.first(where: { format in
                let dimensions = CMVideoFormatDescriptionGetDimensions(format.formatDescription)
                let fps = format.videoSupportedFrameRateRanges.first?.maxFrameRate ?? 0
                return dimensions.width == 1920 && dimensions.height == 1080 && fps >= 60
            }) {
                camera.activeFormat = preferredFormat
                print("Установлен формат: \(preferredFormat)")
            }
            
            camera.unlockForConfiguration()
            
            // Создаем вход после настройки устройства
            return try AVCaptureDeviceInput(device: camera)
            
        } catch {
            print("Ошибка настройки камеры: \(error)")
            return nil
        }
    }
}
```

#### Уровень 5: Множественные входы (например, две камеры одновременно)
На некоторых устройствах можно использовать несколько камер одновременно.

```swift
import UIKit
import AVFoundation

class MultiCameraViewController: InputDemoViewController {
    
    var backCameraInput: AVCaptureDeviceInput?
    var frontCameraInput: AVCaptureDeviceInput?
    
    override func setupSession() {
        captureSession = AVCaptureSession()
        captureSession.sessionPreset = .hd1920x1080
        
        // Пытаемся добавить обе камеры
        addBackCamera()
        addFrontCamera()
        addAudioInput()
        
        // Проверяем, сколько входов добавлено
        let videoInputs = captureSession.inputs.filter { input in
            (input as? AVCaptureDeviceInput)?.device.hasMediaType(.video) ?? false
        }
        
        print("Добавлено видео входов: \(videoInputs.count)")
        
        // Здесь можно добавить выходы для обработки обоих потоков
        
        DispatchQueue.global(qos: .userInitiated).async { [weak self] in
            self?.captureSession.startRunning()
        }
    }
    
    private func addBackCamera() {
        guard let backCamera = AVCaptureDevice.default(.builtInWideAngleCamera, for: .video, position: .back),
              let input = try? AVCaptureDeviceInput(device: backCamera),
              captureSession.canAddInput(input) else {
            print("Не удалось добавить заднюю камеру")
            return
        }
        
        captureSession.addInput(input)
        backCameraInput = input
        print("Задняя камера добавлена")
    }
    
    private func addFrontCamera() {
        guard let frontCamera = AVCaptureDevice.default(.builtInWideAngleCamera, for: .video, position: .front),
              let input = try? AVCaptureDeviceInput(device: frontCamera),
              captureSession.canAddInput(input) else {
            print("Не удалось добавить фронтальную камеру")
            return
        }
        
        captureSession.addInput(input)
        frontCameraInput = input
        print("Фронтальная камера добавлена")
    }
    
    // Получение соединения для конкретного входа
    func getConnectionForInput(_ input: AVCaptureInput, output: AVCaptureOutput) -> AVCaptureConnection? {
        return output.connections.first { connection in
            connection.inputPorts.contains { input.ports.contains($0) }
        }
    }
}
```

#### Уровень 6: Динамическое изменение конфигурации входов
Сложный пример с изменением параметров входа "на лету".

```swift
import UIKit
import AVFoundation

class DynamicInputViewController: InputDemoViewController {
    
    let fpsSlider = UISlider()
    let fpsLabel = UILabel()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupControls()
    }
    
    private func setupControls() {
        fpsLabel.frame = CGRect(x: 20, y: 180, width: 200, height: 30)
        fpsLabel.textColor = .white
        fpsLabel.text = "FPS: 30"
        view.addSubview(fpsLabel)
        
        fpsSlider.frame = CGRect(x: 20, y: 220, width: view.bounds.width - 40, height: 30)
        fpsSlider.minimumValue = 15
        fpsSlider.maximumValue = 60
        fpsSlider.value = 30
        fpsSlider.addTarget(self, action: #selector(fpsChanged), for: .valueChanged)
        view.addSubview(fpsSlider)
    }
    
    @objc func fpsChanged() {
        let fps = Int(fpsSlider.value)
        fpsLabel.text = "FPS: \(fps)"
        configureCameraForFPS(fps)
    }
    
    private func configureCameraForFPS(_ fps: Int) {
        guard let videoInput = videoInput else { return }
        
        let device = videoInput.device
        
        do {
            try device.lockForConfiguration()
            
            // Ищем формат, поддерживающий нужный FPS
            if let format = device.formats.first(where: { format in
                format.videoSupportedFrameRateRanges.contains { range in
                    range.maxFrameRate >= Double(fps) && range.minFrameRate <= Double(fps)
                }
            }) {
                device.activeFormat = format
                device.activeVideoMinFrameDuration = CMTime(value: 1, timescale: CMTimeScale(fps))
                device.activeVideoMaxFrameDuration = CMTime(value: 1, timescale: CMTimeScale(fps))
                print("FPS установлен: \(fps)")
            } else {
                print("Формат с FPS \(fps) не поддерживается")
            }
            
            device.unlockForConfiguration()
            
        } catch {
            print("Ошибка настройки камеры: \(error)")
        }
    }
    
    // Пример: временное переключение на максимальное качество для фото
    func prepareForPhotoCapture() {
        guard let videoInput = videoInput else { return }
        
        let device = videoInput.device
        
        // Сохраняем текущие настройки
        let oldFormat = device.activeFormat
        let oldMinFPS = device.activeVideoMinFrameDuration
        let oldMaxFPS = device.activeVideoMaxFrameDuration
        
        do {
            try device.lockForConfiguration()
            
            // Ищем формат с максимальным разрешением
            if let highResFormat = device.formats.max(by: { format1, format2 in
                let dim1 = CMVideoFormatDescriptionGetDimensions(format1.formatDescription)
                let dim2 = CMVideoFormatDescriptionGetDimensions(format2.formatDescription)
                return (dim1.width * dim1.height) < (dim2.width * dim2.height)
            }) {
                device.activeFormat = highResFormat
                // Устанавливаем низкий FPS для фото
                device.activeVideoMinFrameDuration = CMTime(value: 1, timescale: 5)
                device.activeVideoMaxFrameDuration = CMTime(value: 1, timescale: 5)
                
                // После фото можно вернуть старые настройки
                DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) { [weak self] in
                    self?.restoreCameraSettings(oldFormat: oldFormat, 
                                                oldMinFPS: oldMinFPS, 
                                                oldMaxFPS: oldMaxFPS)
                }
            }
            
            device.unlockForConfiguration()
            
        } catch {
            print("Ошибка временной настройки: \(error)")
        }
    }
    
    private func restoreCameraSettings(oldFormat: AVCaptureDevice.Format, 
                                       oldMinFPS: CMTime, 
                                       oldMaxFPS: CMTime) {
        guard let device = videoInput?.device else { return }
        
        do {
            try device.lockForConfiguration()
            device.activeFormat = oldFormat
            device.activeVideoMinFrameDuration = oldMinFPS
            device.activeVideoMaxFrameDuration = oldMaxFPS
            device.unlockForConfiguration()
        } catch {
            print("Ошибка восстановления: \(error)")
        }
    }
}
```

#### Уровень 7: Кастомный вход из файла (AVCaptureFileInput)
Редкий случай, но полезно знать.

```swift
import UIKit
import AVFoundation
import AVKit

class FileInputViewController: UIViewController {
    
    var captureSession: AVCaptureSession!
    var playerLayer: AVPlayerLayer?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Создаем сессию
        captureSession = AVCaptureSession()
        
        // Создаем вход из файла
        if let fileInput = createFileInput() {
            if captureSession.canAddInput(fileInput) {
                captureSession.addInput(fileInput)
                print("Файловый вход добавлен")
            }
        }
        
        // Добавляем выход для предпросмотра
        let previewOutput = AVCaptureVideoDataOutput()
        if captureSession.canAddOutput(previewOutput) {
            captureSession.addOutput(previewOutput)
        }
        
        // Запускаем сессию
        DispatchQueue.global(qos: .userInitiated).async { [weak self] in
            self?.captureSession.startRunning()
        }
    }
    
    private func createFileInput() -> AVCaptureFileInput? {
        // Путь к видеофайлу в бандле
        guard let videoURL = Bundle.main.url(forResource: "sample", withExtension: "mp4") else {
            print("Видео не найдено")
            return nil
        }
        
        do {
            let fileInput = try AVCaptureFileInput(url: videoURL)
            return fileInput
        } catch {
            print("Ошибка создания файлового входа: \(error)")
            return nil
        }
    }
}
```

---

### Важные нюансы и Best Practices

#### 1. **Проверка canAddInput**
Всегда проверяйте, можно ли добавить вход в сессию, с помощью `canAddInput`. Это может не получиться из-за ограничений устройства или текущей конфигурации.

```swift
if captureSession.canAddInput(input) {
    captureSession.addInput(input)
} else {
    print("Нельзя добавить этот вход")
}
```

#### 2. **beginConfiguration / commitConfiguration**
При добавлении или удалении нескольких входов (или изменении других параметров) используйте эти методы для атомарности изменений:

```swift
captureSession.beginConfiguration()
// Добавление/удаление входов
captureSession.commitConfiguration()
```

#### 3. **Управление устройствами**
Некоторые настройки устройств (фокус, экспозиция, FPS) требуют блокировки конфигурации:

```swift
do {
    try device.lockForConfiguration()
    // Настройка устройства
    device.unlockForConfiguration()
} catch {
    print("Не удалось заблокировать устройство")
}
```

#### 4. **Порты и их включение**
Можно отключать отдельные порты входа, но это менее эффективно, чем отключение целых выходов или соединений. Обычно лучше использовать `AVCaptureConnection.isEnabled`.

#### 5. **Производительность**
- Добавление множества входов может снизить производительность.
- Не все устройства поддерживают одновременную работу двух камер.
- Высокие FPS и разрешения требуют больше ресурсов.

#### 6. **Обработка ошибок**
При создании `AVCaptureDeviceInput` всегда используйте `try-catch`, так как инициализатор может выбросить ошибку (например, устройство занято).

#### 7. **Проверка доступности устройств**
Перед созданием входа проверяйте, доступно ли устройство:

```swift
let device = AVCaptureDevice.default(.builtInWideAngleCamera, for: .video, position: .back)
guard device != nil else { return }
```

#### 8. **Совместимость с различными устройствами**
На разных устройствах могут быть разные камеры. Используйте `DiscoverySession` для получения всех доступных устройств.

### Итог
**AVCaptureInput** — это фундаментальный компонент AVFoundation, обеспечивающий поступление данных в сессию захвата. Основные выводы:

1.  **AVCaptureDeviceInput** — основной класс для работы с камерой и микрофоном.
2.  **Порты** (`ports`) позволяют получить отдельные потоки данных от одного входа.
3.  **Конфигурация** входов должна быть атомарной с `beginConfiguration`/`commitConfiguration`.
4.  **Управление устройствами** требует блокировки через `lockForConfiguration`.
5.  **Множественные входы** возможны, но с ограничениями производительности и совместимости.

Понимание работы с входами необходимо для любой нетривиальной работы с камерой в iOS.