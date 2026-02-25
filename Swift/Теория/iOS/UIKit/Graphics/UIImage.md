**UIImage** — это класс в **[[UIKit]]**, который представляет **растровое изображение** (или иконку) в [[iOS]]-приложениях.

Это один из самых часто используемых типов в iOS-разработке: от простых картинок в [[UIImageView]] до сложных операций с графикой, аватарками, иконками, водяными знаками, обработкой фото и генерацией изображений.

В 2025–2026 годах `UIImage` остаётся **центральным** классом для работы с растровой графикой в UIKit. В SwiftUI он используется через `Image(uiImage:)`.

### Основные способы создания UIImage (самые актуальные в 2026)

| Способ создания                             | Пример кода                                                                | Когда использовать в 2026           | Рекомендация                 |
| ------------------------------------------- | -------------------------------------------------------------------------- | ----------------------------------- | ---------------------------- |
| Из ресурсов проекта (Assets.xcassets)       | `UIImage(named: "avatar")`                                                 | Почти всегда — основной способ      | ★★★★★ (самый безопасный)     |
| Из SF Symbols (системные иконки)            | `UIImage(systemName: "heart.fill")`                                        | Иконки, кнопки, индикаторы          | ★★★★★                        |
| Из данных ([[Data]]) — сеть, файлы          | `UIImage(data: data)`                                                      | Загрузка по [[URL]], из галереи     | ★★★★☆                        |
| Из файла по пути                            | `UIImage(contentsOfFile: path)`                                            | Редко (лучше через Data)            | ★★☆☆☆                        |
| Из [[CGImage]]                              | `UIImage(cgImage: cgImage)`                                                | После обработки в [[Core Graphics]] | ★★★★☆                        |
| Генерация через [[UIGraphicsImageRenderer]] | `UIGraphicsImageRenderer(size: size).image { ... }`                        | Кастомные аватарки, графики         | ★★★★★ (современный стандарт) |
| Системные цвета / градиенты                 | `UIColor.systemBlue.image(size: CGSize(width: 1, height: 1))` (расширение) | Фоновые изображения                 | ★★★★☆                        |
| Из цвета (расширение)                       | `UIColor.systemPurple.image(size: CGSize(width: 100, height: 100))`        | Однотонные фоны, заглушки           | ★★★★☆                        |

### Полезные расширения (очень популярны в 2026)

```swift
extension UIImage {
    // Из HEX-цвета
    convenience init?(color: UIColor, size: CGSize = CGSize(width: 1, height: 1)) {
        let renderer = UIGraphicsImageRenderer(size: size)
        let image = renderer.image { context in
            color.setFill()
            context.fill(CGRect(origin: .zero, size: size))
        }
        self.init(cgImage: image.cgImage!)
    }
    
    // Из SF Symbol с кастомным цветом и весом
    static func symbol(_ name: String, pointSize: CGFloat = 24, weight: UIImage.SymbolWeight = .medium, tint: UIColor? = nil) -> UIImage? {
        var config = UIImage.SymbolConfiguration(pointSize: pointSize, weight: weight)
        if let tint {
            config = config.applying(UIImage.SymbolConfiguration(paletteColors: [tint]))
        }
        return UIImage(systemName: name, withConfiguration: config)
    }
    
    // Асинхронная загрузка с URL (самый популярный кейс)
    static func from(url: URL, completion: @escaping (UIImage?) -> Void) {
        URLSession.shared.dataTask(with: url) { data, _, error in
            guard let data, error == nil else {
                completion(nil)
                return
            }
            DispatchQueue.main.async {
                completion(UIImage(data: data))
            }
        }.resume()
    }
    
    // Сжатие изображения (JPEG)
    func compressedJPEG(quality: CGFloat = 0.8) -> Data? {
        jpegData(compressionQuality: quality)
    }
}
```

### Работа с режимами рендеринга (очень важно в 2026)

```swift
// Template mode — для динамической смены цвета
let icon = UIImage(systemName: "heart.fill")?
    .withRenderingMode(.alwaysTemplate)
imageView.image = icon
imageView.tintColor = .systemRed

// Original mode — сохраняет оригинальные цвета (для фото, логотипов)
let photo = UIImage(named: "userPhoto")?
    .withRenderingMode(.alwaysOriginal)
```

### Масштабируемые изображения (resizableImage)

```swift
// Для пузырьков чата, кнопок с растяжкой
let bubble = UIImage(named: "chatBubble")?
    .resizableImage(
        withCapInsets: UIEdgeInsets(top: 20, left: 20, bottom: 20, right: 20),
        resizingMode: .stretch
    )
bubbleImageView.image = bubble
```

### Асинхронная загрузка (самый популярный кейс в реальных приложениях)

```swift
// Рекомендуемый способ в 2026
extension UIImageView {
    func load(from url: URL, placeholder: UIImage? = nil) {
        image = placeholder
        
        URLSession.shared.dataTask(with: url) { [weak self] data, _, error in
            guard let data, error == nil else { return }
            DispatchQueue.main.async {
                self?.image = UIImage(data: data)
            }
        }.resume()
    }
}

// Использование
imageView.load(from: URL(string: "https://...")!, placeholder: UIImage(systemName: "photo"))
```

### Лучшие практики UIImage в Swift 2026

- **Предпочитайте** `UIImage(named:)` из **Assets.xcassets** — поддержка тёмной темы, scale, локализации  
- **Для иконок** — используйте **SF Symbols** (`systemName:`) — векторные, адаптивные, поддерживают weight и hierarchy  
- **Для сетевых изображений** — всегда асинхронно + placeholder + кэширование ([[Kingfisher]], [[SDWebImage]], или свой кэш)  
- **Для template-цветов** — всегда `.withRenderingMode(.alwaysTemplate)` + `tintColor`  
- **Для масштабируемых** — `resizableImage(withCapInsets:resizingMode:)` — стандарт для чат-пузырей, кнопок  
- **Генерация** — только через `UIGraphicsImageRenderer` (старый `UIGraphicsBeginImageContext` deprecated)  
- **В SwiftUI** — используйте `Image(uiImage:)` или `Image("assetName")` — UIImage нужен только для межплатформенного кода  
- **Документируйте** — пишите комментарий «UIImage(named:) — иконка из Assets.xcassets с поддержкой тёмной темы и scale @3x»

**Короткий итог 2026**:
> UIImage — это **растровый объект изображения** в iOS.  
> В 2026 году:  
> - основной способ создания — `UIImage(named:)` из Assets + `UIImage(systemName:)`  
> - режимы — `.alwaysTemplate` для цвета, `.alwaysOriginal` для фото  
> - масштабирование — `resizableImage(withCapInsets:)`  
> - загрузка из сети — асинхронно + placeholder  
> Это **самый базовый** и **самый часто встречающийся** класс для работы с картинками в UIKit-приложениях.
