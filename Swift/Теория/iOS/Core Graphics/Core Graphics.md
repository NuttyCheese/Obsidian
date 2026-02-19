**Core Graphics** (также известен как **Quartz 2D**) — это фундаментальный фреймворк Apple для **двумерной графики**, рисования, работы с изображениями, путями, текстом, градиентами, масками и PDF.

В Swift Core Graphics используется в [[UIKit]]/[[AppKit]] для:
- кастомного рисования в `draw(_:)`
- создания масок и сложных UI-форм
- работы с растровыми изображениями ([[CGImage]])
- генерации PDF
- низкоуровневого рендеринга в [[CALayer]], `CGBitmapContext`, `CGPDFContext`

Это **низкоуровневый C [[API]]**, но в [[Swift]] он очень удобно обёрнут и активно используется даже в 2026 году.

### Основные сущности Core Graphics (самые важные в 2026)

| Сущность                     | Тип в Swift       | Что это такое                                       | Самый частый сценарий использования               |
| ---------------------------- | ----------------- | --------------------------------------------------- | ------------------------------------------------- |
| `CGContext`                  | `CGContext`       | Контекст рисования (экран, bitmap, PDF)             | `draw(_:)` в [[UIView]], генерация изображений    |
| [[CGPath]] / `CGMutablePath` | `CGPath`          | Математический путь (линии, кривые, прямоугольники) | `CAShapeLayer.path`, маски, clipping              |
| `CGImage`                    | `CGImage`         | Растровое изображение (пиксели)                     | `CALayer.contents`, обработка изображений         |
| [[CGColor]]                  | `CGColor`         | Цвет в определённом цветовом пространстве           | `CALayer.backgroundColor`, `context.setFillColor` |
| `CGColorSpace`               | `CGColorSpace`    | Цветовое пространство (sRGB, Display P3, CMYK)      | Работа с HDR, правильная цветопередача            |
| `CGGradient`                 | `CGGradient`      | Градиент (линейный/радиальный)                      | Градиентные фоны, кнопки                          |
| `CGPDFContext`               | `CGContext` (PDF) | Контекст для генерации PDF                          | Экспорт отчётов, чеков, документов                |
| `CGBitmapContext`            | `CGContext`       | Контекст для работы с пикселями в памяти            | Генерация изображений, фильтры, пиксельный доступ |

### Самые популярные паттерны Core Graphics в Swift 2026

#### 1. Кастомное рисование в UIView (самый частый случай)

```swift
class CustomView: UIView {
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else { return }
        
        // Прямоугольник с градиентом
        let colors = [UIColor.systemBlue.cgColor, UIColor.systemPurple.cgColor] as CFArray
        guard let gradient = CGGradient(colorsSpace: CGColorSpaceCreateDeviceRGB(),
                                       colors: colors,
                                       locations: [0.0, 1.0]) else { return }
        
        context.drawLinearGradient(gradient,
                                  start: CGPoint(x: 0, y: 0),
                                  end: CGPoint(x: bounds.width, y: bounds.height),
                                  options: [])
        
        // Закруглённый прямоугольник
        let path = UIBezierPath(roundedRect: CGRect(x: 20, y: 20, width: 200, height: 100),
                               cornerRadius: 16).cgPath
        
        context.addPath(path)
        context.setLineWidth(4)
        context.setStrokeColor(UIColor.white.cgColor)
        context.strokePath()
    }
}
```

#### 2. Создание изображения из кода (очень популярно для иконок, аватарок, графиков)

```swift
func generateAvatar(size: CGSize, color: UIColor) -> UIImage? {
    UIGraphicsBeginImageContextWithOptions(size, false, 0.0)
    defer { UIGraphicsEndImageContext() }
    
    guard let context = UIGraphicsGetCurrentContext() else { return nil }
    
    let rect = CGRect(origin: .zero, size: size)
    context.setFillColor(color.cgColor)
    context.fillEllipse(in: rect)
    
    // Добавляем букву
    let paragraphStyle = NSMutableParagraphStyle()
    paragraphStyle.alignment = .center
    
    let attrs: [NSAttributedString.Key: Any] = [
        .font: UIFont.boldSystemFont(ofSize: size.width * 0.6),
        .foregroundColor: UIColor.white,
        .paragraphStyle: paragraphStyle
    ]
    
    let text = NSAttributedString(string: "A", attributes: attrs)
    text.draw(in: rect)
    
    return UIGraphicsGetImageFromCurrentImageContext()
}
```

#### 3. Маска для UIView с помощью CGPath

```swift
let maskPath = UIBezierPath(roundedRect: view.bounds,
                            byRoundingCorners: [.topLeft, .bottomRight],
                            cornerRadii: CGSize(width: 30, height: 30)).cgPath

let maskLayer = CAShapeLayer()
maskLayer.path = maskPath
view.layer.mask = maskLayer
```

### Лучшие практики Core Graphics в Swift 2026

- **Всегда** используйте `UIBezierPath` для создания путей — это гораздо удобнее и читаемее  
- **Получайте контекст** через `UIGraphicsGetCurrentContext()` или `CGBitmapContextCreate`  
- **Для производительности** — минимизируйте вызовы `draw(_:)` — кэшируйте изображения  
- **Для [[SwiftUI]]** — предпочитайте `Canvas`, `Path`, `Shape` — они построены поверх Core Graphics  
- **Для [[Metal]] / Core Image** — используйте `CGImage` как входные данные  
- **Swift 6 strict concurrency** — Core Graphics контекст **не [[Sendable]]** — работайте только на главном потоке  
- **Документируйте** — пишите комментарий «Core Graphics — кастомное рисование закруглённого градиентного фона»

**Короткий итог 2026**:
> Core Graphics — это **низкоуровневый движок 2D-графики** Apple, на котором построены почти все UI-компоненты iOS.  
> В 2026 году:  
> - используйте `UIBezierPath` + `.cgPath` для путей  
> - рисуйте в `draw(_:)` или генерируйте `UIImage` через `UIGraphicsImageRenderer`  
> - для `CALayer` — `contents`, `backgroundColor`, `mask`  
> - для SwiftUI — почти всегда `Path` / `Canvas`, но Core Graphics всё ещё нужен для сложных случаев  
