#uikit #uiimage #jpeg #image-processing #deprecated #ios #swift #legacy

---
### Определение
**UIImageJPEGRepresentation** — это функция из фреймворка [[UIKit]], которая использовалась для преобразования объекта [[UIImage]] в данные формата [[JPEG]] (Joint Photographic Experts Group) . Она возвращала объект [[Data]], содержащий изображение в сжатом формате JPEG, готовое для сохранения в файл, отправки по сети или другой обработки.

Функция была частью оригинального [[API]] для работы с изображениями и широко использовалась в приложениях для [[iOS]] до появления более современных альтернатив.

### Важное примечание о статусе Deprecated
**`UIImageJPEGRepresentation` объявлена устаревшей (deprecated)** начиная с iOS 15.0 (2021 год) . Apple рекомендует использовать вместо нее метод экземпляра **`jpegData(compressionQuality:)`** класса `UIImage`. Это часть общей стратегии Apple по переходу от глобальных C-функций к более типобезопасным и объектно-ориентированным API.

### Зачем это знать iOS-разработчику?
1.  **Поддержка легаси-кода:** Многие существующие проекты и онлайн-примеры до сих пор используют эту функцию. Понимание ее работы необходимо для поддержки старого кода.
2.  **Понимание эволюции API:** Знание того, что было раньше, помогает лучше понимать современные подходы.
3.  **Миграция проектов:** При обновлении старых приложений нужно знать, как правильно заменить устаревшие вызовы.
4.  **Работа с JPEG:** Независимо от API, концепции сжатия JPEG остаются теми же, и понимание параметров качества важно.

---

### Синтаксис и параметры

#### Устаревший вариант (iOS 2.0 - iOS 14.x)
```swift
func UIImageJPEGRepresentation(_ image: UIImage, 
                                _ compressionQuality: CGFloat) -> Data?
```

**Параметры:**
- `image`: Исходное изображение для сжатия.
- `compressionQuality`: Качество сжатия от 0.0 (минимальное качество, максимальное сжатие) до 1.0 (максимальное качество, минимальное сжатие).

**Возвращаемое значение:** Объект `Data`, содержащий JPEG-данные, или [[nil]] в случае ошибки.

#### Современный вариант (iOS 11+)
```swift
func jpegData(compressionQuality: CGFloat) -> Data?
```

---

### Примеры от простого к сложному

#### Уровень 0: Сравнение старого и нового API

```swift
import UIKit

class ImageConversionViewController: UIViewController {
    
    @IBOutlet weak var imageView: UIImageView!
    
    // УСТАРЕВШИЙ способ (не используйте в новых проектах!)
    func convertToJPEGLegacy(image: UIImage, quality: CGFloat) -> Data? {
        return UIImageJPEGRepresentation(image, quality)
    }
    
    // СОВРЕМЕННЫЙ способ (рекомендуемый)
    func convertToJPEGModern(image: UIImage, quality: CGFloat) -> Data? {
        return image.jpegData(compressionQuality: quality)
    }
    
    @IBAction func convertImageTapped(_ sender: UIButton) {
        guard let image = imageView.image else { return }
        
        // Используем современный метод
        if let jpegData = image.jpegData(compressionQuality: 0.8) {
            print("JPEG data size: \(jpegData.count) bytes")
            // Далее можно сохранить или отправить данные
        }
    }
}
```

#### Уровень 1: Сохранение JPEG в файл

```swift
import UIKit

class SaveJPEGViewController: UIViewController {
    
    @IBOutlet weak var imageView: UIImageView!
    
    @IBAction func saveToFileTapped(_ sender: UIButton) {
        guard let image = imageView.image else { return }
        
        // Конвертируем в JPEG с качеством 0.8
        guard let jpegData = image.jpegData(compressionQuality: 0.8) else {
            print("Не удалось создать JPEG данные")
            return
        }
        
        // Создаем URL для сохранения в директории документов
        let documentsDirectory = FileManager.default.urls(for: .documentDirectory, 
                                                          in: .userDomainMask).first!
        let fileURL = documentsDirectory.appendingPathComponent("image_\(Date().timeIntervalSince1970).jpg")
        
        do {
            try jpegData.write(to: fileURL)
            print("JPEG сохранен по пути: \(fileURL.path)")
            
            // Показываем пользователю
            showAlert("Изображение сохранено", message: fileURL.lastPathComponent)
        } catch {
            print("Ошибка сохранения: \(error)")
            showAlert("Ошибка", message: error.localizedDescription)
        }
    }
    
    private func showAlert(_ title: String, message: String) {
        let alert = UIAlertController(title: title, message: message, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
}
```

#### Уровень 2: Сравнение качества JPEG

```swift
import UIKit

class QualityComparisonViewController: UIViewController {
    
    @IBOutlet weak var originalImageView: UIImageView!
    @IBOutlet weak var compressedImageView: UIImageView!
    @IBOutlet weak var qualitySlider: UISlider!
    @IBOutlet weak var sizeLabel: UILabel!
    
    var originalImage: UIImage?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        originalImage = originalImageView.image
    }
    
    @IBAction func qualityChanged(_ sender: UISlider) {
        let quality = CGFloat(sender.value)
        
        guard let image = originalImage else { return }
        
        // Конвертируем с выбранным качеством
        if let jpegData = image.jpegData(compressionQuality: quality) {
            // Создаем изображение из сжатых данных
            let compressedImage = UIImage(data: jpegData)
            compressedImageView.image = compressedImage
            
            // Показываем размер
            let sizeKB = Double(jpegData.count) / 1024.0
            sizeLabel.text = String(format: "Качество: %.2f, Размер: %.1f KB", quality, sizeKB)
        }
    }
    
    @IBAction func compareAllQualitiesTapped(_ sender: UIButton) {
        guard let image = originalImage else { return }
        
        var comparisonText = "Сравнение размеров JPEG:\n"
        
        let qualities: [CGFloat] = [0.1, 0.3, 0.5, 0.7, 0.9, 1.0]
        
        for quality in qualities {
            if let jpegData = image.jpegData(compressionQuality: quality) {
                let sizeKB = Double(jpegData.count) / 1024.0
                comparisonText += String(format: "Качество %.2f: %.1f KB\n", quality, sizeKB)
            }
        }
        
        showAlert("Результаты", message: comparisonText)
    }
    
    private func showAlert(_ title: String, message: String) {
        let alert = UIAlertController(title: title, message: message, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
}
```

#### Уровень 3: Пакетная обработка изображений

```swift
import UIKit

class BatchImageProcessor {
    
    static let shared = BatchImageProcessor()
    
    /// Конвертирует все изображения в массиве в JPEG с заданным качеством
    func convertToJPEG(images: [UIImage], 
                       quality: CGFloat, 
                       progressHandler: ((Float) -> Void)? = nil,
                       completion: @escaping ([Data]) -> Void) {
        
        DispatchQueue.global(qos: .userInitiated).async {
            var jpegDataArray: [Data] = []
            let total = Float(images.count)
            
            for (index, image) in images.enumerated() {
                if let jpegData = image.jpegData(compressionQuality: quality) {
                    jpegDataArray.append(jpegData)
                }
                
                // Сообщаем о прогрессе
                let progress = Float(index + 1) / total
                DispatchQueue.main.async {
                    progressHandler?(progress)
                }
            }
            
            DispatchQueue.main.async {
                completion(jpegDataArray)
            }
        }
    }
    
    /// Сохраняет массив JPEG данных в файлы
    func saveJPEGFiles(_ jpegDataArray: [Data], 
                       baseName: String,
                       completion: @escaping ([URL]) -> Void) {
        
        DispatchQueue.global(qos: .background).async {
            var savedURLs: [URL] = []
            let documentsDirectory = FileManager.default.urls(for: .documentDirectory, 
                                                              in: .userDomainMask).first!
            
            for (index, jpegData) in jpegDataArray.enumerated() {
                let fileURL = documentsDirectory
                    .appendingPathComponent("\(baseName)_\(index + 1).jpg")
                
                do {
                    try jpegData.write(to: fileURL)
                    savedURLs.append(fileURL)
                } catch {
                    print("Ошибка сохранения файла \(index + 1): \(error)")
                }
            }
            
            DispatchQueue.main.async {
                completion(savedURLs)
            }
        }
    }
}

// Использование
class BatchProcessingViewController: UIViewController {
    
    @IBOutlet weak var progressView: UIProgressView!
    @IBOutlet weak var statusLabel: UILabel!
    
    var images: [UIImage] = [] // Допустим, массив изображений
    
    @IBAction func processImagesTapped(_ sender: UIButton) {
        progressView.progress = 0
        statusLabel.text = "Конвертация..."
        
        BatchImageProcessor.shared.convertToJPEG(images: images, quality: 0.7, 
                                                 progressHandler: { [weak self] progress in
            self?.progressView.progress = progress
        }) { [weak self] jpegDataArray in
            self?.statusLabel.text = "Сохранение..."
            
            BatchImageProcessor.shared.saveJPEGFiles(jpegDataArray, baseName: "processed") { urls in
                self?.statusLabel.text = "Готово! Сохранено \(urls.count) файлов"
                self?.progressView.progress = 1.0
                
                for url in urls {
                    print("Сохранен: \(url.path)")
                }
            }
        }
    }
}
```

#### Уровень 4: Оптимизация размера изображения перед конвертацией

```swift
import UIKit
import ImageIO

class ImageOptimizer {
    
    /// Оптимизирует изображение: изменяет размер и качество для достижения целевого размера
    static func optimizeImage(_ image: UIImage, 
                               maxSizeKB: Int = 500,
                               initialQuality: CGFloat = 0.9) -> Data? {
        
        var currentQuality = initialQuality
        var currentImage = image
        var resultData: Data?
        
        // Сначала пробуем уменьшить качество
        while currentQuality > 0.2 {
            if let data = currentImage.jpegData(compressionQuality: currentQuality) {
                if data.count / 1024 <= maxSizeKB {
                    resultData = data
                    break
                }
            }
            currentQuality -= 0.1
        }
        
        // Если качество не помогло, уменьшаем размер изображения
        if resultData == nil {
            var scale: CGFloat = 0.9
            
            while scale > 0.3 {
                let newSize = CGSize(width: image.size.width * scale, 
                                      height: image.size.height * scale)
                
                let renderer = UIGraphicsImageRenderer(size: newSize)
                let resizedImage = renderer.image { context in
                    image.draw(in: CGRect(origin: .zero, size: newSize))
                }
                
                if let data = resizedImage.jpegData(compressionQuality: 0.8) {
                    if data.count / 1024 <= maxSizeKB {
                        resultData = data
                        break
                    }
                }
                
                scale -= 0.1
            }
        }
        
        return resultData
    }
    
    /// Создает thumbnail (миниатюру) изображения
    static func createThumbnail(from image: UIImage, 
                                 maxDimension: CGFloat,
                                 quality: CGFloat = 0.7) -> Data? {
        
        let scale = maxDimension / max(image.size.width, image.size.height)
        let newSize = CGSize(width: image.size.width * scale, 
                              height: image.size.height * scale)
        
        let renderer = UIGraphicsImageRenderer(size: newSize)
        let thumbnail = renderer.image { context in
            image.draw(in: CGRect(origin: .zero, size: newSize))
        }
        
        return thumbnail.jpegData(compressionQuality: quality)
    }
}

// Использование
class OptimizationViewController: UIViewController {
    
    @IBOutlet weak var imageView: UIImageView!
    @IBOutlet weak var infoLabel: UILabel!
    
    @IBAction func optimizeTapped(_ sender: UIButton) {
        guard let image = imageView.image else { return }
        
        // Исходный размер
        if let originalData = image.jpegData(compressionQuality: 1.0) {
            let originalKB = originalData.count / 1024
            infoLabel.text = "Исходный: \(originalKB) KB\n"
        }
        
        // Оптимизация
        if let optimizedData = ImageOptimizer.optimizeImage(image, maxSizeKB: 200) {
            let optimizedKB = optimizedData.count / 1024
            
            infoLabel.text? += "Оптимизированный: \(optimizedKB) KB\n"
            infoLabel.text? += "Экономия: \(100 - (optimizedKB * 100 / (originalData?.count ?? 1) / 1024))%"
            
            // Показываем оптимизированное изображение
            let optimizedImage = UIImage(data: optimizedData)
            imageView.image = optimizedImage
        }
    }
    
    @IBAction func createThumbnailTapped(_ sender: UIButton) {
        guard let image = imageView.image else { return }
        
        if let thumbnailData = ImageOptimizer.createThumbnail(from: image, 
                                                              maxDimension: 300, 
                                                              quality: 0.6) {
            let thumbnailKB = thumbnailData.count / 1024
            infoLabel.text = "Thumbnail: \(thumbnailKB) KB"
            
            let thumbnailImage = UIImage(data: thumbnailData)
            imageView.image = thumbnailImage
        }
    }
}
```

#### Уровень 5: Отправка JPEG на сервер

```swift
import UIKit

class NetworkUploadViewController: UIViewController {
    
    @IBOutlet weak var imageView: UIImageView!
    @IBOutlet weak var progressView: UIProgressView!
    @IBOutlet weak var statusLabel: UILabel!
    
    @IBAction func uploadImageTapped(_ sender: UIButton) {
        guard let image = imageView.image else { return }
        
        statusLabel.text = "Подготовка..."
        progressView.progress = 0
        
        // Конвертируем в JPEG с оптимальным качеством
        guard let jpegData = image.jpegData(compressionQuality: 0.7) else {
            statusLabel.text = "Ошибка конвертации"
            return
        }
        
        statusLabel.text = "Загрузка..."
        uploadJPEGData(jpegData)
    }
    
    private func uploadJPEGData(_ jpegData: Data) {
        // Создаем URL запроса
        guard let url = URL(string: "https://api.example.com/upload") else {
            statusLabel.text = "Неверный URL"
            return
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        
        // Создаем multipart form data
        let boundary = UUID().uuidString
        request.setValue("multipart/form-data; boundary=\(boundary)", forHTTPHeaderField: "Content-Type")
        
        var body = Data()
        
        // Добавляем данные изображения
        body.append("--\(boundary)\r\n".data(using: .utf8)!)
        body.append("Content-Disposition: form-data; name=\"image\"; filename=\"image.jpg\"\r\n".data(using: .utf8)!)
        body.append("Content-Type: image/jpeg\r\n\r\n".data(using: .utf8)!)
        body.append(jpegData)
        body.append("\r\n".data(using: .utf8)!)
        body.append("--\(boundary)--\r\n".data(using: .utf8)!)
        
        request.httpBody = body
        
        // Создаем задачу загрузки с прогрессом
        let session = URLSession(configuration: .default, 
                                 delegate: self, 
                                 delegateQueue: OperationQueue.main)
        
        let task = session.uploadTask(with: request, from: body) { [weak self] data, response, error in
            if let error = error {
                self?.statusLabel.text = "Ошибка: \(error.localizedDescription)"
                return
            }
            
            self?.statusLabel.text = "Загрузка завершена!"
            self?.progressView.progress = 1.0
        }
        
        task.resume()
    }
}

extension NetworkUploadViewController: URLSessionTaskDelegate {
    
    func urlSession(_ session: URLSession, 
                    task: URLSessionTask, 
                    didSendBodyData bytesSent: Int64, 
                    totalBytesSent: Int64, 
                    totalBytesExpectedToSend: Int64) {
        
        let progress = Float(totalBytesSent) / Float(totalBytesExpectedToSend)
        progressView.progress = progress
        statusLabel.text = String(format: "Загрузка: %.1f%%", progress * 100)
    }
}
```

#### Уровень 6: Конвертация между форматами с JPEG как промежуточным

```swift
import UIKit

class ImageFormatConverter {
    
    /// Конвертирует HEIC в JPEG
    static func convertHEICToJPEG(heicData: Data, 
                                   quality: CGFloat = 0.8) -> Data? {
        guard let image = UIImage(data: heicData) else { return nil }
        return image.jpegData(compressionQuality: quality)
    }
    
    /// Конвертирует PNG в JPEG (теряет прозрачность!)
    static func convertPNGToJPEG(pngData: Data,
                                  quality: CGFloat = 0.8,
                                  backgroundColor: UIColor = .white) -> Data? {
        
        guard let image = UIImage(data: pngData) else { return nil }
        
        // PNG может содержать прозрачность, которую JPEG не поддерживает
        // Нужно отрисовать на фоне
        let renderer = UIGraphicsImageRenderer(size: image.size)
        let jpegImage = renderer.image { context in
            // Заливаем фон
            backgroundColor.setFill()
            context.fill(CGRect(origin: .zero, size: image.size))
            
            // Рисуем изображение поверх
            image.draw(at: .zero)
        }
        
        return jpegImage.jpegData(compressionQuality: quality)
    }
    
    /// Конвертирует RAW в JPEG (требуется Core Image)
    static func convertRAWToJPEG(rawURL: URL,
                                  quality: CGFloat = 0.9) -> Data? {
        
        let ciImage = CIImage(contentsOf: rawURL)
        let context = CIContext()
        
        guard let cgImage = context.createCGImage(ciImage!, from: ciImage!.extent) else {
            return nil
        }
        
        let image = UIImage(cgImage: cgImage)
        return image.jpegData(compressionQuality: quality)
    }
    
    /// Пакетная конвертация всех изображений в папке
    static func convertAllImagesInFolder(folderURL: URL,
                                          toJPEGWithQuality quality: CGFloat,
                                          completion: @escaping ([URL]) -> Void) {
        
        DispatchQueue.global(qos: .userInitiated).async {
            let fileManager = FileManager.default
            var convertedURLs: [URL] = []
            
            do {
                let files = try fileManager.contentsOfDirectory(at: folderURL, 
                                                                includingPropertiesForKeys: nil)
                
                for file in files {
                    let ext = file.pathExtension.lowercased()
                    
                    if ext == "heic" || ext == "png" || ext == "jpg" || ext == "jpeg" {
                        if let data = try? Data(contentsOf: file),
                           let image = UIImage(data: data) {
                            
                            let jpegURL = file.deletingPathExtension()
                                .appendingPathExtension("jpg")
                            
                            if let jpegData = image.jpegData(compressionQuality: quality) {
                                try? jpegData.write(to: jpegURL)
                                convertedURLs.append(jpegURL)
                            }
                        }
                    }
                }
                
                DispatchQueue.main.async {
                    completion(convertedURLs)
                }
                
            } catch {
                print("Ошибка: \(error)")
                DispatchQueue.main.async {
                    completion([])
                }
            }
        }
    }
}

// Использование
class FormatConversionViewController: UIViewController {
    
    @IBAction func convertHEICToJPEGTapped(_ sender: UIButton) {
        // Допустим, у нас есть HEIC данные
        let heicData = Data() // Ваши HEIC данные
        
        if let jpegData = ImageFormatConverter.convertHEICToJPEG(heicData: heicData) {
            print("JPEG размер: \(jpegData.count / 1024) KB")
        }
    }
    
    @IBAction func convertPNGWithBackgroundTapped(_ sender: UIButton) {
        let pngData = Data() // Ваши PNG данные
        
        if let jpegData = ImageFormatConverter.convertPNGToJPEG(pngData: pngData, 
                                                                 backgroundColor: .blue) {
            print("JPEG с синим фоном готов")
        }
    }
}
```

#### Уровень 7: Миграция проекта с UIImageJPEGRepresentation

```swift
import UIKit

// MARK: - Совместимость с легаси
extension UIImage {
    
    /// Обертка для обратной совместимости (симулирует старую функцию)
    @available(*, deprecated, message: "Используйте jpegData(compressionQuality:) вместо UIImageJPEGRepresentation")
    func legacyJPEGRepresentation(quality: CGFloat) -> Data? {
        return self.jpegData(compressionQuality: quality)
    }
}

// MARK: - Пример рефакторинга легаси-кода

// ДО рефакторинга (старый код)
class LegacyImageManager {
    
    func saveImageLegacy(_ image: UIImage) -> Bool {
        // Старый способ - НЕ РЕКОМЕНДУЕТСЯ
        guard let data = UIImageJPEGRepresentation(image, 0.8) else {
            return false
        }
        
        let url = getDocumentsDirectory().appendingPathComponent("image.jpg")
        try? data.write(to: url)
        return true
    }
    
    private func getDocumentsDirectory() -> URL {
        return FileManager.default.urls(for: .documentDirectory, 
                                        in: .userDomainMask)[0]
    }
}

// ПОСЛЕ рефакторинга (новый код)
class ModernImageManager {
    
    func saveImageModern(_ image: UIImage) -> Bool {
        // Новый способ - РЕКОМЕНДУЕТСЯ
        guard let data = image.jpegData(compressionQuality: 0.8) else {
            return false
        }
        
        let url = getDocumentsDirectory().appendingPathComponent("image.jpg")
        try? data.write(to: url)
        return true
    }
    
    private func getDocumentsDirectory() -> URL {
        return FileManager.default.urls(for: .documentDirectory, 
                                        in: .userDomainMask)[0]
    }
}

// MARK: - Поиск и замена в проекте
// При миграции проекта можно использовать регулярные выражения для замены:
// Найти: UIImageJPEGRepresentation\((\w+),\s*([\d.]+)\)
// Заменить: $1.jpegData(compressionQuality: $2)
```

#### Уровень 8: Сравнение производительности

```swift
import UIKit

class PerformanceComparisonViewController: UIViewController {
    
    @IBOutlet weak var resultTextView: UITextView!
    
    @IBAction func runPerformanceTestTapped(_ sender: UIButton) {
        guard let testImage = UIImage(named: "test_large_image") else {
            resultTextView.text = "Тестовое изображение не найдено"
            return
        }
        
        var results = "Тестирование производительности\n\n"
        
        // Тест 1: UIImageJPEGRepresentation (если доступно)
        if #available(iOS, deprecated: 15.0) {
            let start1 = CFAbsoluteTimeGetCurrent()
            for _ in 0..<10 {
                _ = UIImageJPEGRepresentation(testImage, 0.8)
            }
            let end1 = CFAbsoluteTimeGetCurrent()
            results += String(format: "UIImageJPEGRepresentation: %.4f сек\n", (end1 - start1))
        } else {
            results += "UIImageJPEGRepresentation: недоступно\n"
        }
        
        // Тест 2: jpegData
        let start2 = CFAbsoluteTimeGetCurrent()
        for _ in 0..<10 {
            _ = testImage.jpegData(compressionQuality: 0.8)
        }
        let end2 = CFAbsoluteTimeGetCurrent()
        results += String(format: "jpegData: %.4f сек\n", (end2 - start2))
        
        // Тест 3: Разные качества
        results += "\nСравнение размеров:\n"
        let qualities: [CGFloat] = [0.1, 0.5, 0.9]
        for quality in qualities {
            if let data = testImage.jpegData(compressionQuality: quality) {
                let sizeKB = Double(data.count) / 1024.0
                results += String(format: "  Качество %.1f: %.1f KB\n", quality, sizeKB)
            }
        }
        
        resultTextView.text = results
    }
}
```

---

### UIImageJPEGRepresentation vs jpegData

| Характеристика | UIImageJPEGRepresentation (устаревшая) | jpegData (современная) |
|----------------|----------------------------------------|------------------------|
| **Тип** | Глобальная C-функция | Метод экземпляра UIImage |
| **Доступность** | iOS 2.0 - 14.x | iOS 11+ |
| **Статус** | Deprecated (iOS 15+) | Актуальный |
| **Безопасность типов** | Низкая | Высокая (вызывается у объекта) |
| **Readability** | Менее очевидно, к какому объекту относится | Понятно, что это метод UIImage |
| **Производительность** | Аналогична | Аналогична |
| **Поддержка** | Не рекомендуется для новых проектов | Рекомендуется |

---

### Важные нюансы и Best Practices

#### 1. **Всегда проверяйте возвращаемое значение**
Метод может вернуть `nil`, например, если изображение не может быть преобразовано в JPEG.

```swift
guard let jpegData = image.jpegData(compressionQuality: 0.8) else {
    print("Не удалось создать JPEG")
    return
}
```

#### 2. **Выбирайте правильное качество**
- **0.8 - 0.9**: Хороший баланс для фотографий.
- **0.5 - 0.7**: Для превью и thumbnail.
- **1.0**: Для максимального качества (но файлы будут большими).
- **< 0.3**: Только для очень маленьких изображений (сильные артефакты).

#### 3. **Учитывайте память**
JPEG данные могут быть большими. При работе с множеством изображений используйте оптимизацию и background queues.

#### 4. **JPEG не поддерживает прозрачность**
При конвертации PNG с прозрачностью в JPEG, прозрачные области станут черными (или нужно задать фон).

#### 5. **Альтернативы для прозрачности**
Если нужна прозрачность, используйте PNG: `image.pngData()`.

#### 6. **Миграция кода**
При обновлении старых проектов используйте поиск и замену с регулярными выражениями для автоматической миграции.

### Итог
**UIImageJPEGRepresentation** — это историческая функция, которая долгое время была стандартом для конвертации изображений в JPEG. Хотя она объявлена устаревшей, понимание ее работы необходимо для поддержки легаси-кода. Для всех новых проектов используйте современный метод **`jpegData(compressionQuality:)`**, который предоставляет тот же функционал в более чистом и типобезопасном виде.

Ключевые навыки: конвертация UIImage в JPEG, выбор оптимального качества, оптимизация размера изображений, работа с файловой системой, отправка JPEG на сервер, миграция легаси-кода.