#uikit #uiimage #png #image-processing #deprecated #ios #swift #legacy #transparency

---
### Определение
**UIImagePNGRepresentation** — это функция из фреймворка [[UIKit]], которая использовалась для преобразования объекта [[UIImage]] в данные формата PNG (Portable Network Graphics) . Она возвращала объект [[Data]], содержащий изображение в формате [[PNG]] со сжатием без потерь, готовое для сохранения в файл, отправки по сети или другой обработки.

Функция была частью оригинального [[API]] для работы с изображениями и широко использовалась в приложениях для [[iOS]] до появления более современных альтернатив.

### Важное примечание о статусе Deprecated
**`UIImagePNGRepresentation` объявлена устаревшей (deprecated)** начиная с iOS 15.0 (2021 год) . Apple рекомендует использовать вместо нее метод экземпляра **`pngData()`** класса `UIImage`. Это часть общей стратегии Apple по переходу от глобальных C-функций к более типобезопасным и объектно-ориентированным API.

### Зачем это знать iOS-разработчику?
1.  **Поддержка легаси-кода:** Многие существующие проекты и онлайн-примеры до сих пор используют эту функцию. Понимание ее работы необходимо для поддержки старого кода.
2.  **Сохранение прозрачности:** PNG — основной формат для изображений с прозрачностью (альфа-каналом).
3.  **Сжатие без потерь:** В отличие от JPEG, PNG сохраняет все детали изображения, что важно для графики, текста и скриншотов.
4.  **Миграция проектов:** При обновлении старых приложений нужно знать, как правильно заменить устаревшие вызовы.

---

### PNG vs JPEG: Ключевые отличия

| Характеристика | PNG | JPEG |
|----------------|-----|------|
| **Сжатие** | Без потерь | С потерями |
| **Прозрачность** | Поддерживает (альфа-канал) | Не поддерживает |
| **Фотографии** | Большой размер | Маленький размер |
| **Графика/Текст** | Отлично (без артефактов) | Плохо (артефакты) |
| **Логотипы, иконки** | Идеально | Не подходит |
| **Скриншоты** | Идеально | Есть артефакты |
| **Размер файла** | Больше | Меньше |

### Синтаксис и параметры

#### Устаревший вариант (iOS 2.0 - iOS 14.x)
```swift
func UIImagePNGRepresentation(_ image: UIImage) -> Data?
```

**Параметры:**
- `image`: Исходное изображение для преобразования.

**Возвращаемое значение:** Объект `Data`, содержащий PNG-данные, или `nil` в случае ошибки.

#### Современный вариант (iOS 11+)
```swift
func pngData() -> Data?
```

---

### Примеры от простого к сложному

#### Уровень 0: Сравнение старого и нового API

```swift
import UIKit

class PNGConversionViewController: UIViewController {
    
    @IBOutlet weak var imageView: UIImageView!
    
    // УСТАРЕВШИЙ способ (не используйте в новых проектах!)
    func convertToPNGLegacy(image: UIImage) -> Data? {
        return UIImagePNGRepresentation(image)
    }
    
    // СОВРЕМЕННЫЙ способ (рекомендуемый)
    func convertToPNGModern(image: UIImage) -> Data? {
        return image.pngData()
    }
    
    @IBAction func convertImageTapped(_ sender: UIButton) {
        guard let image = imageView.image else { return }
        
        // Используем современный метод
        if let pngData = image.pngData() {
            print("PNG data size: \(pngData.count) bytes")
            // Далее можно сохранить или отправить данные
        }
    }
}
```

#### Уровень 1: Сохранение PNG с прозрачностью в файл

```swift
import UIKit

class SavePNGViewController: UIViewController {
    
    @IBOutlet weak var imageView: UIImageView!
    
    @IBAction func saveToFileTapped(_ sender: UIButton) {
        guard let image = imageView.image else { return }
        
        // Конвертируем в PNG (с сохранением прозрачности)
        guard let pngData = image.pngData() else {
            print("Не удалось создать PNG данные")
            return
        }
        
        // Создаем URL для сохранения в директории документов
        let documentsDirectory = FileManager.default.urls(for: .documentDirectory, 
                                                          in: .userDomainMask).first!
        let fileURL = documentsDirectory.appendingPathComponent("image_\(Date().timeIntervalSince1970).png")
        
        do {
            try pngData.write(to: fileURL)
            print("PNG сохранен по пути: \(fileURL.path)")
            
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

#### Уровень 2: PNG vs [[JPEG]] — демонстрация прозрачности и качества

```swift
import UIKit

class PNGvsJPEGViewController: UIViewController {
    
    @IBOutlet weak var originalImageView: UIImageView!
    @IBOutlet weak var pngImageView: UIImageView!
    @IBOutlet weak var jpegImageView: UIImageView!
    @IBOutlet weak var infoLabel: UILabel!
    
    func demonstrateTransparency() {
        // Создаем изображение с прозрачностью
        let renderer = UIGraphicsImageRenderer(size: CGSize(width: 200, height: 200))
        let transparentImage = renderer.image { context in
            // Рисуем красный круг с прозрачностью
            UIColor.red.withAlphaComponent(0.5).setFill()
            context.cgContext.fillEllipse(in: CGRect(x: 20, y: 20, width: 100, height: 100))
            
            // Рисуем синий квадрат с прозрачностью
            UIColor.blue.withAlphaComponent(0.5).setFill()
            context.cgContext.fill(CGRect(x: 80, y: 80, width: 100, height: 100))
        }
        
        originalImageView.image = transparentImage
        
        // Конвертируем в PNG (сохраняет прозрачность)
        if let pngData = transparentImage.pngData() {
            let pngImage = UIImage(data: pngData)
            pngImageView.image = pngImage
            
            let pngSizeKB = Double(pngData.count) / 1024.0
            infoLabel.text = String(format: "PNG: %.1f KB", pngSizeKB)
        }
        
        // Конвертируем в JPEG (теряет прозрачность)
        if let jpegData = transparentImage.jpegData(compressionQuality: 0.9) {
            let jpegImage = UIImage(data: jpegData)
            jpegImageView.image = jpegImage
            
            let jpegSizeKB = Double(jpegData.count) / 1024.0
            infoLabel.text? += String(format: "\nJPEG: %.1f KB", jpegSizeKB)
        }
    }
    
    @IBAction func compareTapped(_ sender: UIButton) {
        demonstrateTransparency()
    }
}
```

#### Уровень 3: Пакетная обработка и оптимизация PNG

```swift
import UIKit
import ImageIO
import MobileCoreServices

class PNGProcessor {
    
    /// Оптимизирует PNG (без потери качества, но иногда можно уменьшить размер)
    static func optimizePNG(image: UIImage) -> Data? {
        guard let originalData = image.pngData() else { return nil }
        
        let originalSizeKB = originalData.count / 1024
        print("Оригинальный PNG размер: \(originalSizeKB) KB")
        
        // PNG уже сжат без потерь, но можно попробовать
        // пересохранить с удалением метаданных
        
        // Создаем источник изображения
        guard let source = CGImageSourceCreateWithData(originalData as CFData, nil),
              let uti = CGImageSourceGetType(source) else {
            return originalData
        }
        
        // Настройки для оптимизации
        let options: [String: Any] = [
            kCGImageDestinationLossyCompressionQuality as String: 1.0, // Без потерь
            kCGImagePropertyHasAlpha as String: true // Сохраняем альфа
        ]
        
        let destinationData = NSMutableData()
        guard let destination = CGImageDestinationCreateWithData(destinationData, uti, 1, nil) else {
            return originalData
        }
        
        CGImageDestinationAddImageFromSource(destination, source, 0, options as CFDictionary)
        
        if CGImageDestinationFinalize(destination) {
            let optimizedSizeKB = destinationData.length / 1024
            print("Оптимизированный PNG размер: \(optimizedSizeKB) KB")
            
            // Если оптимизация не помогла, возвращаем оригинал
            if destinationData.length < originalData.count {
                return destinationData as Data
            }
        }
        
        return originalData
    }
    
    /// Конвертирует массив UIImage в PNG данные
    static func batchConvertToPNG(images: [UIImage], 
                                   progressHandler: ((Float) -> Void)? = nil,
                                   completion: @escaping ([Data]) -> Void) {
        
        DispatchQueue.global(qos: .userInitiated).async {
            var pngDataArray: [Data] = []
            let total = Float(images.count)
            
            for (index, image) in images.enumerated() {
                if let pngData = image.pngData() {
                    pngDataArray.append(pngData)
                }
                
                let progress = Float(index + 1) / total
                DispatchQueue.main.async {
                    progressHandler?(progress)
                }
            }
            
            DispatchQueue.main.async {
                completion(pngDataArray)
            }
        }
    }
    
    /// Сохраняет массив PNG данных в файлы
    static func savePNGFiles(_ pngDataArray: [Data], 
                              baseName: String,
                              completion: @escaping ([URL]) -> Void) {
        
        DispatchQueue.global(qos: .background).async {
            var savedURLs: [URL] = []
            let documentsDirectory = FileManager.default.urls(for: .documentDirectory, 
                                                              in: .userDomainMask).first!
            
            for (index, pngData) in pngDataArray.enumerated() {
                let fileURL = documentsDirectory
                    .appendingPathComponent("\(baseName)_\(index + 1).png")
                
                do {
                    try pngData.write(to: fileURL)
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
class BatchPNGViewController: UIViewController {
    
    @IBOutlet weak var progressView: UIProgressView!
    @IBOutlet weak var statusLabel: UILabel!
    
    var images: [UIImage] = [] // Массив изображений
    
    @IBAction func processImagesTapped(_ sender: UIButton) {
        progressView.progress = 0
        statusLabel.text = "Конвертация..."
        
        PNGProcessor.batchConvertToPNG(images: images, progressHandler: { [weak self] progress in
            self?.progressView.progress = progress
        }) { [weak self] pngDataArray in
            self?.statusLabel.text = "Сохранение..."
            
            PNGProcessor.savePNGFiles(pngDataArray, baseName: "processed") { urls in
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

#### Уровень 4: Работа с прозрачностью и альфа-каналом

```swift
import UIKit

class TransparencyViewController: UIViewController {
    
    @IBOutlet weak var imageView: UIImageView!
    
    // Создание изображения с градиентной прозрачностью
    func createTransparentGradientImage(size: CGSize) -> UIImage? {
        let renderer = UIGraphicsImageRenderer(size: size)
        
        let image = renderer.image { context in
            // Создаем градиент с прозрачностью
            let colors = [
                UIColor.red.cgColor,
                UIColor.blue.withAlphaComponent(0.5).cgColor,
                UIColor.green.withAlphaComponent(0.0).cgColor
            ]
            
            let colorSpace = CGColorSpaceCreateDeviceRGB()
            let locations: [CGFloat] = [0.0, 0.5, 1.0]
            
            if let gradient = CGGradient(colorsSpace: colorSpace,
                                         colors: colors as CFArray,
                                         locations: locations) {
                
                context.cgContext.drawLinearGradient(gradient,
                                                     start: CGPoint(x: 0, y: 0),
                                                     end: CGPoint(x: size.width, y: size.height),
                                                     options: [])
            }
            
            // Рисуем текст с прозрачностью
            let text = "PNG"
            let attributes: [NSAttributedString.Key: Any] = [
                .font: UIFont.boldSystemFont(ofSize: 60),
                .foregroundColor: UIColor.white.withAlphaComponent(0.7)
            ]
            
            let textSize = text.size(withAttributes: attributes)
            let textPoint = CGPoint(x: (size.width - textSize.width) / 2,
                                   y: (size.height - textSize.height) / 2)
            
            text.draw(at: textPoint, withAttributes: attributes)
        }
        
        return image
    }
    
    @IBAction func createAndSaveTransparentPNGTapped(_ sender: UIButton) {
        guard let image = createTransparentGradientImage(size: CGSize(width: 300, height: 300)) else {
            return
        }
        
        imageView.image = image
        
        // Сохраняем как PNG (сохранит прозрачность)
        if let pngData = image.pngData() {
            let sizeKB = Double(pngData.count) / 1024.0
            print("PNG с прозрачностью: \(sizeKB) KB")
            
            // Сохраняем в файл
            let documentsDirectory = FileManager.default.urls(for: .documentDirectory, 
                                                              in: .userDomainMask).first!
            let fileURL = documentsDirectory.appendingPathComponent("transparent_gradient.png")
            
            do {
                try pngData.write(to: fileURL)
                showAlert("Сохранено", message: "Файл: \(fileURL.lastPathComponent)")
            } catch {
                showAlert("Ошибка", message: error.localizedDescription)
            }
        }
    }
    
    private func showAlert(_ title: String, message: String) {
        let alert = UIAlertController(title: title, message: message, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
}
```

#### Уровень 5: Извлечение информации о PNG

```swift
import UIKit
import ImageIO

class PNGInfoExtractor {
    
    struct PNGInfo {
        let size: CGSize
        let hasAlpha: Bool
        let colorSpaceName: String
        let bitDepth: Int
        let fileSizeKB: Double
        let isInterlaced: Bool
        let gamma: CGFloat?
    }
    
    static func extractPNGInfo(from data: Data) -> PNGInfo? {
        guard let image = UIImage(data: data),
              let cgImage = image.cgImage,
              let source = CGImageSourceCreateWithData(data as CFData, nil) else {
            return nil
        }
        
        // Получаем свойства изображения
        let properties = CGImageSourceCopyPropertiesAtIndex(source, 0, nil) as? [CFString: Any]
        
        // Размер
        let size = image.size
        
        // Наличие альфа-канала
        let alphaInfo = cgImage.alphaInfo
        let hasAlpha = alphaInfo != .none && 
                       alphaInfo != .noneSkipFirst && 
                       alphaInfo != .noneSkipLast
        
        // Цветовое пространство
        let colorSpaceName = cgImage.colorSpace?.name as? String ?? "Unknown"
        
        // Битность
        let bitDepth = cgImage.bitsPerPixel / 4
        
        // Размер файла
        let fileSizeKB = Double(data.count) / 1024.0
        
        // Чересстрочная развертка (interlacing)
        let isInterlaced = properties?[kCGImagePropertyPNGInterlaceType] as? Bool ?? false
        
        // Гамма
        let gamma = properties?[kCGImagePropertyPNGGamma] as? CGFloat
        
        return PNGInfo(
            size: size,
            hasAlpha: hasAlpha,
            colorSpaceName: colorSpaceName,
            bitDepth: bitDepth,
            fileSizeKB: fileSizeKB,
            isInterlaced: isInterlaced,
            gamma: gamma
        )
    }
    
    static func printPNGInfo(_ info: PNGInfo) {
        print("=== PNG Информация ===")
        print("Размер: \(info.size.width) x \(info.size.height)")
        print("Альфа-канал: \(info.hasAlpha ? "Да" : "Нет")")
        print("Цветовое пространство: \(info.colorSpaceName)")
        print("Глубина цвета: \(info.bitDepth) бит на канал")
        print("Размер файла: \(String(format: "%.1f", info.fileSizeKB)) KB")
        print("Чересстрочная: \(info.isInterlaced ? "Да" : "Нет")")
        if let gamma = info.gamma {
            print("Гамма: \(gamma)")
        }
    }
}

// Использование
class PNGInfoViewController: UIViewController {
    
    @IBOutlet weak var imageView: UIImageView!
    @IBOutlet weak var infoTextView: UITextView!
    
    @IBAction func analyzePNGTapped(_ sender: UIButton) {
        guard let image = imageView.image,
              let pngData = image.pngData() else { return }
        
        if let info = PNGInfoExtractor.extractPNGInfo(from: pngData) {
            var infoText = "=== PNG Информация ===\n"
            infoText += "Размер: \(info.size.width) x \(info.size.height)\n"
            infoText += "Альфа-канал: \(info.hasAlpha ? "Да" : "Нет")\n"
            infoText += "Цветовое пространство: \(info.colorSpaceName)\n"
            infoText += "Глубина цвета: \(info.bitDepth) бит\n"
            infoText += String(format: "Размер файла: %.1f KB\n", info.fileSizeKB)
            infoText += "Чересстрочная: \(info.isInterlaced ? "Да" : "Нет")\n"
            
            infoTextView.text = infoText
        }
    }
}
```

#### Уровень 6: Конвертация между форматами с сохранением прозрачности

```swift
import UIKit

class FormatConverter {
    
    /// Конвертирует JPEG в PNG (добавляет возможность прозрачности)
    static func convertJPEGToPNG(jpegData: Data) -> Data? {
        guard let image = UIImage(data: jpegData) else { return nil }
        
        // JPEG не имеет прозрачности, но мы можем создать PNG
        // с тем же содержимым
        return image.pngData()
    }
    
    /// Конвертирует HEIC в PNG (сохраняет прозрачность, если есть)
    static func convertHEICToPNG(heicData: Data) -> Data? {
        guard let image = UIImage(data: heicData) else { return nil }
        return image.pngData()
    }
    
    /// Конвертирует PDF страницу в PNG с прозрачностью
    static func convertPDFPageToPNG(pdfURL: URL, 
                                      pageIndex: Int = 0,
                                      size: CGSize) -> Data? {
        
        guard let document = CGPDFDocument(pdfURL as CFURL),
              let page = document.page(at: pageIndex + 1) else { return nil }
        
        let renderer = UIGraphicsImageRenderer(size: size)
        let image = renderer.image { context in
            // Белый фон, если не нужна прозрачность, или можно сделать прозрачным
            UIColor.white.setFill()
            context.fill(CGRect(origin: .zero, size: size))
            
            // Рисуем PDF страницу
            context.cgContext.translateBy(x: 0, y: size.height)
            context.cgContext.scaleBy(x: 1, y: -1)
            
            let pdfRect = page.getBoxRect(.mediaBox)
            let scale = min(size.width / pdfRect.width, size.height / pdfRect.height)
            
            context.cgContext.scaleBy(x: scale, y: scale)
            context.cgContext.drawPDFPage(page)
        }
        
        return image.pngData()
    }
    
    /// Конвертирует векторный PDF в PNG с заданным размером
    static func convertVectorPDFToPNG(pdfData: Data,
                                        size: CGSize) -> Data? {
        
        guard let provider = CGDataProvider(data: pdfData as CFData),
              let pdf = CGPDFDocument(provider) else { return nil }
        
        let renderer = UIGraphicsImageRenderer(size: size)
        let image = renderer.image { context in
            // Прозрачный фон
            context.cgContext.clear(CGRect(origin: .zero, size: size))
            
            guard let page = pdf.page(at: 1) else { return }
            
            let pdfRect = page.getBoxRect(.mediaBox)
            let scale = min(size.width / pdfRect.width, size.height / pdfRect.height)
            
            context.cgContext.translateBy(x: (size.width - pdfRect.width * scale) / 2,
                                          y: (size.height - pdfRect.height * scale) / 2)
            context.cgContext.scaleBy(x: scale, y: scale)
            context.cgContext.drawPDFPage(page)
        }
        
        return image.pngData()
    }
}

// Использование
class FormatConversionViewController: UIViewController {
    
    @IBAction func convertJPEGToPNGTapped(_ sender: UIButton) {
        // Допустим, у нас есть JPEG данные
        let jpegData = Data() // Ваши JPEG данные
        
        if let pngData = FormatConverter.convertJPEGToPNG(jpegData: jpegData) {
            print("PNG размер: \(pngData.count / 1024) KB")
        }
    }
}
```

#### Уровень 7: Создание PNG с метаданными

```swift
import UIKit
import ImageIO
import MobileCoreServices

class PNGMetadataCreator {
    
    /// Создает PNG с пользовательскими метаданными
    static func createPNGWithMetadata(image: UIImage,
                                        metadata: [CFString: Any]) -> Data? {
        
        guard let cgImage = image.cgImage else { return nil }
        
        let data = NSMutableData()
        guard let destination = CGImageDestinationCreateWithData(data, 
                                                                  UTType.png.identifier as CFString, 
                                                                  1, 
                                                                  nil) else {
            return nil
        }
        
        // Добавляем метаданные
        CGImageDestinationAddImage(destination, cgImage, metadata as CFDictionary)
        
        if CGImageDestinationFinalize(destination) {
            return data as Data
        }
        
        return nil
    }
    
    /// Создает PNG с EXIF данными
    static func createPNGWithEXIF(image: UIImage,
                                    exifData: [CFString: Any]) -> Data? {
        
        var metadata: [CFString: Any] = [:]
        metadata[kCGImagePropertyExifDictionary] = exifData
        
        return createPNGWithMetadata(image: image, metadata: metadata)
    }
    
    /// Создает PNG с IPTC данными
    static func createPNGWithIPTC(image: UIImage,
                                    iptcData: [CFString: Any]) -> Data? {
        
        var metadata: [CFString: Any] = [:]
        metadata[kCGImagePropertyIPTCDictionary] = iptcData
        
        return createPNGWithMetadata(image: image, metadata: metadata)
    }
    
    /// Создает PNG с GPS координатами
    static func createPNGWithGPS(image: UIImage,
                                   latitude: Double,
                                   longitude: Double) -> Data? {
        
        let gpsData: [CFString: Any] = [
            kCGImagePropertyGPSLatitude: latitude,
            kCGImagePropertyGPSLongitude: longitude,
            kCGImagePropertyGPSLatitudeRef: latitude >= 0 ? "N" : "S",
            kCGImagePropertyGPSLongitudeRef: longitude >= 0 ? "E" : "W"
        ]
        
        var metadata: [CFString: Any] = [:]
        metadata[kCGImagePropertyGPSDictionary] = gpsData
        
        return createPNGWithMetadata(image: image, metadata: metadata)
    }
}

// Использование
class MetadataViewController: UIViewController {
    
    @IBOutlet weak var imageView: UIImageView!
    
    @IBAction func createPNGWithMetadataTapped(_ sender: UIButton) {
        guard let image = imageView.image else { return }
        
        // Создаем метаданные
        let exif: [CFString: Any] = [
            kCGImagePropertyExifDateTimeOriginal: Date().description,
            kCGImagePropertyExifPixelXDimension: Int(image.size.width),
            kCGImagePropertyExifPixelYDimension: Int(image.size.height)
        ]
        
        let iptc: [CFString: Any] = [
            kCGImagePropertyIPTCCaption: "Тестовое изображение",
            kCGImagePropertyIPTCCopyrightNotice: "© 2026"
        ]
        
        var metadata: [CFString: Any] = [:]
        metadata[kCGImagePropertyExifDictionary] = exif
        metadata[kCGImagePropertyIPTCDictionary] = iptc
        
        // Создаем PNG с метаданными
        if let pngData = PNGMetadataCreator.createPNGWithMetadata(image: image, metadata: metadata) {
            let sizeKB = Double(pngData.count) / 1024.0
            print("PNG с метаданными: \(sizeKB) KB")
            
            // Сохраняем
            let documentsDirectory = FileManager.default.urls(for: .documentDirectory, 
                                                              in: .userDomainMask).first!
            let fileURL = documentsDirectory.appendingPathComponent("metadata_image.png")
            
            do {
                try pngData.write(to: fileURL)
                showAlert("Сохранено", message: "PNG с метаданными сохранен")
            } catch {
                showAlert("Ошибка", message: error.localizedDescription)
            }
        }
    }
    
    private func showAlert(_ title: String, message: String) {
        let alert = UIAlertController(title: title, message: message, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
}
```

#### Уровень 8: Миграция проекта с UIImagePNGRepresentation

```swift
import UIKit

// MARK: - Совместимость с легаси
extension UIImage {
    
    /// Обертка для обратной совместимости (симулирует старую функцию)
    @available(*, deprecated, message: "Используйте pngData() вместо UIImagePNGRepresentation")
    func legacyPNGRepresentation() -> Data? {
        return self.pngData()
    }
}

// MARK: - Пример рефакторинга легаси-кода

// ДО рефакторинга (старый код)
class LegacyImageManager {
    
    func saveImageLegacy(_ image: UIImage) -> Bool {
        // Старый способ - НЕ РЕКОМЕНДУЕТСЯ
        guard let data = UIImagePNGRepresentation(image) else {
            return false
        }
        
        let url = getDocumentsDirectory().appendingPathComponent("image.png")
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
        guard let data = image.pngData() else {
            return false
        }
        
        let url = getDocumentsDirectory().appendingPathComponent("image.png")
        try? data.write(to: url)
        return true
    }
    
    private func getDocumentsDirectory() -> URL {
        return FileManager.default.urls(for: .documentDirectory, 
                                        in: .userDomainMask)[0]
    }
}

// MARK: - Автоматическая миграция
// Регулярное выражение для поиска и замены:
// Найти: UIImagePNGRepresentation\((\w+)\)
// Заменить: $1.pngData()
```

#### Уровень 9: Сравнение производительности PNG и JPEG

```swift
import UIKit

class PerformanceComparisonViewController: UIViewController {
    
    @IBOutlet weak var resultTextView: UITextView!
    
    @IBAction func runPerformanceTestTapped(_ sender: UIButton) {
        guard let testImage = UIImage(named: "test_large_image") else {
            resultTextView.text = "Тестовое изображение не найдено"
            return
        }
        
        var results = "Тестирование производительности PNG vs JPEG\n\n"
        
        // Тест PNG
        let startPNG = CFAbsoluteTimeGetCurrent()
        var pngSize = 0
        for _ in 0..<5 {
            if let data = testImage.pngData() {
                pngSize = data.count
            }
        }
        let endPNG = CFAbsoluteTimeGetCurrent()
        results += String(format: "PNG (5 итераций): %.4f сек\n", (endPNG - startPNG))
        results += String(format: "PNG размер: %.1f KB\n\n", Double(pngSize) / 1024.0)
        
        // Тест JPEG с разным качеством
        let qualities: [CGFloat] = [1.0, 0.9, 0.8, 0.5, 0.3]
        
        results += "JPEG с разным качеством:\n"
        for quality in qualities {
            let startJPEG = CFAbsoluteTimeGetCurrent()
            var jpegSize = 0
            for _ in 0..<5 {
                if let data = testImage.jpegData(compressionQuality: quality) {
                    jpegSize = data.count
                }
            }
            let endJPEG = CFAbsoluteTimeGetCurrent()
            
            results += String(format: "  Качество %.2f: %.4f сек, ", quality, (endJPEG - startJPEG))
            results += String(format: "размер: %.1f KB\n", Double(jpegSize) / 1024.0)
        }
        
        // Тест на изображении с прозрачностью
        results += "\nТест на изображении с прозрачностью:\n"
        let transparentImage = createTransparentTestImage()
        
        if let pngTransparentData = transparentImage.pngData() {
            let pngSizeKB = Double(pngTransparentData.count) / 1024.0
            results += String(format: "  PNG с прозрачностью: %.1f KB\n", pngSizeKB)
        }
        
        if let jpegTransparentData = transparentImage.jpegData(compressionQuality: 0.9) {
            let jpegSizeKB = Double(jpegTransparentData.count) / 1024.0
            results += String(format: "  JPEG (теряет прозрачность): %.1f KB\n", jpegSizeKB)
        }
        
        resultTextView.text = results
    }
    
    private func createTransparentTestImage() -> UIImage {
        let renderer = UIGraphicsImageRenderer(size: CGSize(width: 200, height: 200))
        return renderer.image { context in
            UIColor.red.withAlphaComponent(0.5).setFill()
            context.cgContext.fillEllipse(in: CGRect(x: 50, y: 50, width: 100, height: 100))
        }
    }
}
```

---

### UIImagePNGRepresentation vs pngData

| Характеристика | UIImagePNGRepresentation (устаревшая) | pngData (современная) |
|----------------|----------------------------------------|----------------------|
| **Тип** | Глобальная C-функция | Метод экземпляра UIImage |
| **Доступность** | iOS 2.0 - 14.x | iOS 11+ |
| **Статус** | Deprecated (iOS 15+) | Актуальный |
| **Безопасность типов** | Низкая | Высокая (вызывается у объекта) |
| **Readability** | Менее очевидно, к какому объекту относится | Понятно, что это метод UIImage |
| **Производительность** | Аналогична | Аналогична |
| **Поддержка прозрачности** | Да | Да |
| **Поддержка** | Не рекомендуется для новых проектов | Рекомендуется |

---

### Важные нюансы и Best Practices

#### 1. **Всегда проверяйте возвращаемое значение**
Метод может вернуть `nil`, если изображение не может быть преобразовано в PNG.

```swift
guard let pngData = image.pngData() else {
    print("Не удалось создать PNG")
    return
}
```

#### 2. **PNG сохраняет прозрачность**
В отличие от JPEG, PNG сохраняет альфа-канал. Используйте PNG для:
- Логотипов и иконок
- Изображений с прозрачными областями
- Скриншотов интерфейса
- Графиков и диаграмм

#### 3. **Размер файла**
PNG файлы обычно больше JPEG. Для фотографий предпочтительнее JPEG.

#### 4. **Оптимизация PNG**
Для уменьшения размера PNG можно:
- Использовать инструменты типа TinyPNG
- Уменьшить глубину цвета (PNG-8 вместо PNG-24)
- Удалить метаданные

#### 5. **Цветовое пространство**
PNG поддерживает различные цветовые пространства. Убедитесь, что используете правильное для вашего случая.

#### 6. **Миграция кода**
При обновлении старых проектов используйте поиск и замену с регулярными выражениями.

### Итог
**UIImagePNGRepresentation** — это историческая функция, которая долгое время была стандартом для конвертации изображений в PNG. Хотя она объявлена устаревшей, понимание ее работы необходимо для поддержки легаси-кода и работы с изображениями, требующими прозрачности. Для всех новых проектов используйте современный метод **`pngData()`**, который предоставляет тот же функционал в более чистом и типобезопасном виде.

Ключевые навыки: конвертация UIImage в PNG, сохранение прозрачности, работа с альфа-каналом, оптимизация PNG, извлечение метаданных, сравнение с JPEG, миграция легаси-кода.