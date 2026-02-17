**Rendering blocks** (или **блоки рендеринга**) — это современный и рекомендуемый способ выполнения операций рисования в [[UIKit]], начиная с iOS 10 (2016 год). Они пришли на смену устаревшим функциям `UIGraphicsBeginImageContext` / `UIGraphicsEndImageContext`.

Это замыкания ([[closure]]), которые передаются в методы рендереров (`UIGraphicsImageRenderer`, `UIGraphicsPDFRenderer`, `UIGraphicsImageRendererFormat` и т.д.), и внутри них выполняется весь код рисования.

### Почему rendering blocks — это стандарт 2026 года

| Старый способ (до iOS 10)                  | Новый способ (rendering blocks)                     | Преимущества нового подхода |
|--------------------------------------------|-----------------------------------------------------|-----------------------------|
| `UIGraphicsBeginImageContext(size)`        | `UIGraphicsImageRenderer(size:).image { ctx in ... }` | Автоматическое управление контекстом |
| `UIGraphicsGetCurrentContext()`            | Контекст передаётся в замыкание как параметр       | Нет риска неправильного контекста |
| `UIGraphicsEndImageContext()`              | Контекст автоматически завершается после замыкания | Нет утечек памяти |
| Масштабирование и цветовые пространства вручную | Поддержка `@2x`, `@3x`, HDR, wide color автоматически | Поддержка всех современных экранов |
| Не thread-safe                             | Thread-safe (можно вызывать с background)           | Безопасно для асинхронного рендеринга |

### Основные рендереры и их методы (2026 актуально)

| Рендерер                              | Основное назначение                          | Самый частый метод                  | Возвращает          |
|---------------------------------------|----------------------------------------------|-------------------------------------|---------------------|
| `UIGraphicsImageRenderer`             | Создание `UIImage`                           | `.image(actions:)`                  | `UIImage`           |
| `UIGraphicsPDFRenderer`               | Создание PDF-документов                      | `.pdfData(actions:)` / `.writePDF(to:actions:)` | `Data` или файл     |
| `UIGraphicsImageRendererFormat`       | Настройка формата (scale, color space, opacity) | —                                   | —                   |

### Самый популярный и рекомендуемый паттерн 2026 года

#### 1. Создание простого изображения (иконка, аватар, placeholder)

```swift
func createRoundedAvatarPlaceholder(size: CGSize, color: UIColor) -> UIImage {
    let renderer = UIGraphicsImageRenderer(size: size)
    
    return renderer.image { ctx in
        // Фон
        color.setFill()
        ctx.fill(CGRect(origin: .zero, size: size))
        
        // Скруглённый круг
        let inset = size.width * 0.1
        let circleRect = CGRect(x: inset, y: inset, width: size.width - inset * 2, height: size.height - inset * 2)
        UIBezierPath(ovalIn: circleRect).addClip()
        
        UIColor.white.setFill()
        ctx.fill(circleRect)
        
        // Иконка человека (SF Symbol)
        let config = UIImage.SymbolConfiguration(pointSize: size.width * 0.5, weight: .medium)
        if let personImage = UIImage(systemName: "person.fill", withConfiguration: config) {
            let imageRect = CGRect(x: (size.width - personImage.size.width) / 2,
                                  y: (size.height - personImage.size.height) / 2,
                                  width: personImage.size.width,
                                  height: personImage.size.height)
            personImage.draw(in: imageRect)
        }
    }
}
```

#### 2. Создание PDF с несколькими страницами

```swift
func generateInvoicePDF(data: InvoiceData) -> Data {
    let pageWidth: CGFloat = 595.0   // A4 в 72 dpi
    let pageHeight: CGFloat = 842.0
    let renderer = UIGraphicsPDFRenderer(bounds: CGRect(x: 0, y: 0, width: pageWidth, height: pageHeight))
    
    return renderer.pdfData { context in
        for page in 1...data.numberOfPages {
            context.beginPage()
            
            // Заголовок
            let title = "Invoice #\(page)"
            let attrs: [NSAttributedString.Key: Any] = [.font: UIFont.boldSystemFont(ofSize: 24)]
            title.draw(at: CGPoint(x: 50, y: 50), withAttributes: attrs)
            
            // Таблица, логотип, подпись — всё внутри замыкания
            // ...
        }
    }
}
```

#### 3. Рендеринг с поддержкой scale и HDR (современный экран)

```swift
let format = UIGraphicsImageRendererFormat()
format.scale = UIScreen.main.scale          // @2x / @3x
format.preferredRange = .automatic          // HDR, wide color если экран поддерживает
format.opaque = false

let renderer = UIGraphicsImageRenderer(size: size, format: format)
let image = renderer.image { ctx in
    // код рисования
}
```

### Лучшие практики rendering blocks в Swift 2026

- **Всегда используй `UIGraphicsImageRenderer`** вместо старых `UIGraphicsBeginImageContext`  
- **Передавай format** с правильным `scale` и `preferredRange` для поддержки современных экранов  
- **Не делай тяжёлые вычисления внутри замыкания** — это выполняется синхронно на главном потоке  
- **Асинхронный рендеринг** — если нужно рендерить много/сложно, используй `DispatchQueue.global().async` + `UIGraphicsImageRenderer`  
- **[[@MainActor]]** — если рендеришь на главном потоке (для UI)  
- **Документируйте** — пиши комментарий «[[UIGraphicsImageRenderer]] — генерация placeholder-аватарки с градиентом»

**Короткий девиз 2026**:
> Rendering blocks — это **единственный современный** способ рисовать изображения, PDF и графику в UIKit.  
> `UIGraphicsImageRenderer.image { ctx in ... }` — твой основной инструмент.  
> Забудь старые `UIGraphicsBeginImageContext` — они устарели с 2016 года и могут привести к утечкам памяти.
