**`CGColor`** — это структура (C-структура) из **[[Core Graphics]]** (Quartz), представляющая **цвет в пространстве цветов** (color space) с набором компонентов (обычно RGBA, CMYK, grayscale и т.д.).

В Swift это **не Swift-тип**, а **C-структура**, которая активно используется в UIKit/AppKit для работы с цветами на уровне **Core Graphics / Core Animation** (CALayer, CGContext и т.д.).

### Ключевые характеристики CGColor (актуально на 2026 год)

| Характеристика | Описание                                                | Важные замечания 2026                          |
| -------------- | ------------------------------------------------------- | ---------------------------------------------- |
| Тип            | `CGColor` (C-структура, не объект)                      | Не является [[NSObject]] / [[AnyObject]]       |
| Компоненты     | Массив `CGFloat` (обычно 4 для RGBA)                    | Длина зависит от color space                   |
| Color Space    | `CGColorSpace` (sRGB, P3, CMYK, DeviceRGB и т.д.)       | Чаще всего `CGColorSpaceCreateDeviceRGB()`     |
| Alpha          | Последний компонент — прозрачность (0.0–1.0)            | Всегда присутствует в большинстве color spaces |
| Создание       | `CGColorCreate`, `CGColorCreateCopy`, `UIColor.cgColor` | Почти всегда через `UIColor` / `NSColor`       |
| Swift-обёртка  | `CGColor` → `UIColor` / `Color` (SwiftUI)               | Рекомендуется работать через Swift-типы        |

### Самые частые способы получения CGColor в 2026 году

#### 1. Из [[UIColor]] (самый популярный и рекомендуемый)

```swift
let uiColor = UIColor.systemBlue
let cgColor = uiColor.cgColor

let layer = CALayer()
layer.backgroundColor = cgColor  // ← здесь нужен CGColor
```

#### 2. Прямое создание CGColor (редко, но иногда нужно)

```swift
let colorSpace = CGColorSpaceCreateDeviceRGB()
let components: [CGFloat] = [0.0, 0.5, 1.0, 1.0] // R G B A
if let cgColor = CGColor(colorSpace: colorSpace, components: components) {
    view.layer.backgroundColor = cgColor
}
```

#### 3. В [[SwiftUI]] → UIColor → CGColor (через bridging)

```swift
import SwiftUI

let swiftUIColor = Color.blue
let uiColor = UIColor(swiftUIColor)
let cgColor = uiColor.cgColor
```

#### 4. CGColor → UIColor (обратное преобразование)

```swift
let cgColor: CGColor = UIColor.systemPurple.cgColor
let uiColor = UIColor(cgColor: cgColor)
let swiftUIColor = Color(uiColor)
```

### Самые популярные сценарии использования CGColor в 2026

1. **CALayer** — фон, border, shadow, gradient  
   ```swift
   layer.backgroundColor = UIColor.systemTeal.cgColor
   layer.borderColor = UIColor.red.cgColor
   ```

2. **CGContext** — рисование в Core Graphics  
   ```swift
   context.setFillColor(UIColor.systemGreen.cgColor)
   context.fill(rect)
   ```

3. **CAGradientLayer**  
   ```swift
   let gradient = CAGradientLayer()
   gradient.colors = [UIColor.systemBlue.cgColor, UIColor.systemPurple.cgColor]
   ```

4. **CAShapeLayer**  
   ```swift
   let shapeLayer = CAShapeLayer()
   shapeLayer.strokeColor = UIColor.systemOrange.cgColor
   ```

5. **Custom UIView drawing**  
   ```swift
   override func draw(_ rect: CGRect) {
       guard let context = UIGraphicsGetCurrentContext() else { return }
       context.setFillColor(UIColor.systemIndigo.cgColor)
       context.fill(rect)
   }
   ```

### Лучшие практики работы с CGColor в Swift 2026

- **Всегда** создавайте CGColor **через UIColor** / `Color` — это самый безопасный и читаемый способ  
- **Никогда** не создавайте CGColor вручную через `CGColorCreate`, если можно обойтись `UIColor.cgColor`  
- **Для SwiftUI** — предпочитайте `Color` — `CGColor` нужен только при работе с CALayer  
- **Для градиентов и теней** — всегда используйте `CGColor` (CALayer требует именно его)  
- **Swift 6 strict concurrency** — `CGColor` полностью безопасен ([[Sendable]]), но [[CALayer]] и контекст — только на главном потоке  
- **Документируйте** — пишите комментарий «CGColor — цвет для CALayer.backgroundColor (systemBlue)»

**Короткий итог 2026**:
> `CGColor` — это **низкоуровневое представление цвета** для Core Graphics и Core Animation.  
> В 2026 году:  
> - получайте его **всегда** через `UIColor.cgColor` или `Color` → `UIColor` → `cgColor`  
> - используйте для `CALayer`, `CGContext`, `CAGradientLayer`  
> - не создавайте вручную, если не работаете с нестандартным color space  
> Это **мост** между высокоуровневым UIColor/Color и низкоуровневым рисованием  
