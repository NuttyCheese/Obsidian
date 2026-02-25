**UIGraphicsImageRenderer** — это современный и рекомендуемый класс в [[UIKit]] (с [[iOS]] 10, 2016 год), который заменил устаревшие `UIGraphicsBeginImageContext` / `UIGraphicsEndImageContext`.

Он используется для **программного создания изображений** (UIImage) в коде: генерация иконок, аватарок, графиков, водяных знаков, скриншотов, PDF-страниц, обработки изображений и т.д.

### Почему UIGraphicsImageRenderer — стандарт 2025–2026 годов

| Сравнение с UIGraphicsBeginImageContext | UIGraphicsBeginImageContext (старый) | UIGraphicsImageRenderer (новый)            | Выигрыш |
| --------------------------------------- | ------------------------------------ | ------------------------------------------ | ------- |
| Поддержка scale (Retina, @2x, @3x)      | Нужно вручную передавать scale       | Автоматически учитывает scale экрана       | ★★★★★   |
| Поддержка sRGB / Display P3             | Часто получался неправильный цвет    | Корректно работает с цветовыми профилями   | ★★★★★   |
| Thread-safety                           | Небезопасен в фоне                   | Полностью безопасен (особенно с format)    | ★★★★★   |
| Возможность создания PDF                | Требует отдельного CGPDFContext      | Есть `UIGraphicsPDFRenderer` (родственный) | ★★★★☆   |
| Лёгкость использования                  | Много boilerplate, defer обязателен  | Чистый [[API]], замыкание, меньше ошибок   | ★★★★★   |
| Современность                           | Deprecated в документации Apple      | Рекомендуемый способ с 2016 года           | ★★★★★   |

### Основные способы использования UIGraphicsImageRenderer

#### 1. Самый частый паттерн — создание UIImage из кода

```swift
func generateAvatar(size: CGSize, background: UIColor, text: String) -> UIImage? {
    let renderer = UIGraphicsImageRenderer(size: size)
    
    return renderer.image { context in
        // Заливка фона
        background.setFill()
        context.fill(CGRect(origin: .zero, size: size))
        
        // Текст по центру
        let paragraphStyle = NSMutableParagraphStyle()
        paragraphStyle.alignment = .center
        
        let attrs: [NSAttributedString.Key: Any] = [
            .font: UIFont.boldSystemFont(ofSize: size.width * 0.5),
            .foregroundColor: .white,
            .paragraphStyle: paragraphStyle
        ]
        
        let textRect = CGRect(x: 0, y: (size.height - size.width * 0.5) / 2,
                              width: size.width, height: size.width * 0.5)
        
        text.draw(in: textRect, withAttributes: attrs)
    }
}
```

#### 2. Создание с поддержкой scale (Retina, @3x)

```swift
func snapshot(view: UIView) -> UIImage? {
    let format = UIGraphicsImageRendererFormat()
    format.scale = UIScreen.main.scale          // учитывает Retina
    format.opaque = false                       // прозрачность
    format.preferredRange = .automatic          // sRGB / Display P3
    
    let renderer = UIGraphicsImageRenderer(bounds: view.bounds, format: format)
    
    return renderer.image { context in
        view.layer.render(in: context.cgContext)
    }
}
```

#### 3. Создание градиентного изображения

```swift
func gradientImage(size: CGSize, colors: [UIColor]) -> UIImage? {
    let format = UIGraphicsImageRendererFormat.default()
    format.scale = UIScreen.main.scale
    
    let renderer = UIGraphicsImageRenderer(size: size, format: format)
    
    return renderer.image { context in
        guard let cgContext = context.cgContext else { return }
        
        let gradient = CGGradient(
            colorsSpace: CGColorSpaceCreateDeviceRGB(),
            colors: colors.map { $0.cgColor } as CFArray,
            locations: nil
        )
        
        cgContext.drawLinearGradient(
            gradient!,
            start: CGPoint(x: 0, y: 0),
            end: CGPoint(x: size.width, y: size.height),
            options: []
        )
    }
}
```

#### 4. Создание PDF (UIGraphicsPDFRenderer — близкий родственник)

```swift
func generatePDF(from views: [UIView]) -> Data? {
    let format = UIGraphicsPDFRendererFormat()
    format.documentInfo = [kCGPDFContextCreator as String: "MyApp"]
    
    let renderer = UIGraphicsPDFRenderer(bounds: CGRect(origin: .zero, size: CGSize(width: 595, height: 842)), format: format)
    
    let data = renderer.pdfData { context in
        for view in views {
            context.beginPage()
            view.layer.render(in: context.cgContext)
        }
    }
    
    return data
}
```

### Лучшие практики UIGraphicsImageRenderer в Swift 2026

- **Всегда** используйте `UIGraphicsImageRenderer` вместо `UIGraphicsBeginImageContext` — старый API deprecated и опасен в фоне  
- **Задавайте** `format.scale = UIScreen.main.scale` — особенно важно для Retina и @3x  
- **Используйте** `format.opaque = false` для прозрачных изображений  
- **Для правильных цветов** — оставляйте `preferredRange = .automatic` (адаптируется к экрану)  
- **Для многошаговой отрисовки** — используйте замыкание renderer.image { ctx in ... }  
- **Для производительности** — избегайте создания изображений в main thread при большом размере  
- **Для [[SwiftUI]]** — используйте `Image(uiImage:)` после генерации в `Task` или `onAppear`  
- **Документируйте** — пишите комментарий «UIGraphicsImageRenderer — генерация аватарки с градиентом и текстом по центру (Retina-ready)»

**Короткий итог 2026**:
> UIGraphicsImageRenderer — это **современный и единственный рекомендуемый** способ создавать UIImage программно.  
> В 2026 году:  
> - заменяет устаревший `UIGraphicsBeginImageContext`  
> - ключевые фичи: правильный scale, поддержка P3/sRGB, thread-safety  
> - самый частый паттерн — `renderer.image { context in ... }`  
> - идеально для: аватарок, иконок, графиков, водяных знаков, скриншотов, PDF  
