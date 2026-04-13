#core-graphics #cggradient #gradient #drawing #ios #swift #animation

---

## CGGradient — Градиенты в Core Graphics

### Определение

**`CGGradient`** — это объект в [[Core Graphics]], который представляет **градиентную заливку** — плавный переход между двумя или более цветами вдоль заданного направления (линейного или радиального). В отличие от [[CAGradientLayer]] (который работает на уровне Core Animation), `CGGradient` используется для низкоуровневого рисования в [[CGContext]], например, в `draw(_:)` методах или при создании кастомных [[UIView]].

### Зачем это знать iOS-разработчику?

1.  **Кастомное рисование:** Создание сложных градиентных фонов в `draw(_:)`.
2.  **Анимация:** Плавное изменение цветов градиента во времени.
3.  **Маски:** Комбинирование градиентов с масками для создания эффектов.
4.  **Тени и объём:** Использование градиентов для имитации и теней.
5.  **Диаграммы и графики:** Заливка областей под графиками градиентом.

---

### Типы градиентов

| Тип | Описание |
|---|---|
| **Линейный (Linear)** | Цвета изменяются вдоль прямой линии между двумя точками. |
| **Радиальный (Radial)** | Цвета изменяются от центральной точки к внешнему радиусу (или наоборот). |

---

### Создание CGGradient

#### 1. **С помощью цветового пространства и массива цветов**

```swift
import UIKit

func createSimpleGradient() -> CGGradient? {
    let colors = [UIColor.red.cgColor, UIColor.blue.cgColor]
    let colorSpace = CGColorSpaceCreateDeviceRGB()
    let locations: [CGFloat] = [0.0, 1.0]
    
    return CGGradient(colorsSpace: colorSpace, colors: colors as CFArray, locations: locations)
}
```

#### 2. **С помощью компонентов (низкоуровневый)**

```swift
func createGradientWithComponents() -> CGGradient? {
    let colorSpace = CGColorSpaceCreateDeviceRGB()
    let components: [CGFloat] = [
        1.0, 0.0, 0.0, 1.0,  // Red
        0.0, 0.0, 1.0, 1.0   // Blue
    ]
    let locations: [CGFloat] = [0.0, 1.0]
    
    return CGGradient(colorSpace: colorSpace, colorComponents: components,
                      locations: locations, count: 2)
}
```

#### 3. **Утилитарная функция для создания градиента из массива [[UIColor]]**

```swift
extension CGGradient {
    static func linear(colors: [UIColor], locations: [CGFloat]? = nil) -> CGGradient? {
        let cgColors = colors.map { $0.cgColor }
        let colorSpace = cgColors.first?.colorSpace ?? CGColorSpaceCreateDeviceRGB()
        
        return CGGradient(colorsSpace: colorSpace, colors: cgColors as CFArray,
                         locations: locations)
    }
    
    static func radial(colors: [UIColor], locations: [CGFloat]? = nil) -> CGGradient? {
        return linear(colors: colors, locations: locations)
    }
}

// Использование
let gradient = CGGradient.linear(colors: [.red, .green, .blue])
```

---

### Рисование градиента в [[CGContext]]

#### 1. **Линейный градиент**

```swift
class LinearGradientView: UIView {
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else { return }
        
        let colors = [UIColor.red.cgColor, UIColor.blue.cgColor]
        let colorSpace = CGColorSpaceCreateDeviceRGB()
        let locations: [CGFloat] = [0.0, 1.0]
        
        guard let gradient = CGGradient(colorsSpace: colorSpace,
                                        colors: colors as CFArray,
                                        locations: locations) else { return }
        
        let startPoint = CGPoint(x: rect.minX, y: rect.midY)
        let endPoint = CGPoint(x: rect.maxX, y: rect.midY)
        
        context.drawLinearGradient(gradient, start: startPoint, end: endPoint,
                                   options: [])
    }
}
```

#### 2. **Радиальный градиент**

```swift
class RadialGradientView: UIView {
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else { return }
        
        let colors = [UIColor.yellow.cgColor, UIColor.orange.cgColor, UIColor.red.cgColor]
        let colorSpace = CGColorSpaceCreateDeviceRGB()
        let locations: [CGFloat] = [0.0, 0.5, 1.0]
        
        guard let gradient = CGGradient(colorsSpace: colorSpace,
                                        colors: colors as CFArray,
                                        locations: locations) else { return }
        
        let center = CGPoint(x: rect.midX, y: rect.midY)
        let radius = min(rect.width, rect.height) / 2
        
        context.drawRadialGradient(gradient, startCenter: center, startRadius: 0,
                                   endCenter: center, endRadius: radius,
                                   options: [])
    }
}
```

#### 3. **Градиент с опциями (extend start/end)**

```swift
// Опции расширения градиента
let options: CGGradientDrawingOptions = [.drawsBeforeStart, .drawsAfterEnd]

context.drawLinearGradient(gradient, start: startPoint, end: endPoint, options: options)
```

| Опция | Описание |
|---|---|
| **`.drawsBeforeStart`** | Заливает область до начальной точки первым цветом. |
| **`.drawsAfterEnd`** | Заливает область после конечной точки последним цветом. |

---

### Примеры сложных градиентов

#### 1. **Многоцветный линейный градиент**

```swift
class RainbowGradientView: UIView {
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else { return }
        
        let colors: [CGColor] = [
            UIColor.red.cgColor,
            UIColor.orange.cgColor,
            UIColor.yellow.cgColor,
            UIColor.green.cgColor,
            UIColor.blue.cgColor,
            UIColor.purple.cgColor
        ]
        
        let locations: [CGFloat] = [0.0, 0.2, 0.4, 0.6, 0.8, 1.0]
        let colorSpace = CGColorSpaceCreateDeviceRGB()
        
        guard let gradient = CGGradient(colorsSpace: colorSpace,
                                        colors: colors as CFArray,
                                        locations: locations) else { return }
        
        context.drawLinearGradient(gradient,
                                   start: CGPoint(x: rect.minX, y: rect.minY),
                                   end: CGPoint(x: rect.maxX, y: rect.maxY),
                                   options: [])
    }
}
```

#### 2. **Диагональный градиент**

```swift
class DiagonalGradientView: UIView {
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else { return }
        
        let gradient = CGGradient.linear(colors: [.systemBlue, .systemGreen])
        
        context.drawLinearGradient(gradient!,
                                   start: CGPoint(x: rect.minX, y: rect.minY),
                                   end: CGPoint(x: rect.maxX, y: rect.maxY),
                                   options: [])
    }
}
```

#### 3. **Градиент в форме круга (с маской)**

```swift
class MaskedGradientView: UIView {
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else { return }
        
        // Создаём маску (круг)
        let maskPath = UIBezierPath(ovalIn: CGRect(x: 50, y: 50, width: 200, height: 200))
        context.addPath(maskPath.cgPath)
        context.clip()
        
        // Рисуем градиент внутри маски
        let gradient = CGGradient.linear(colors: [.red, .blue])
        context.drawLinearGradient(gradient!,
                                   start: CGPoint(x: rect.minX, y: rect.midY),
                                   end: CGPoint(x: rect.maxX, y: rect.midY),
                                   options: [])
    }
}
```

---

### CGGradient vs CAGradientLayer

| Характеристика         | `CGGradient`                    | `CAGradientLayer`                    |
| ---------------------- | ------------------------------- | ------------------------------------ |
| **Уровень**            | [[Core Graphics]] (низкий)      | [[Core Animation]] (высокий)         |
| **Использование**      | `draw(_:)`, кастомное рисование | Добавление как `sublayer`            |
| **Производительность** | Выше (один раз отрисовать)      | Ниже (перерисовывается при анимации) |
| **Анимация**           | Сложно (требует перерисовки)    | Легко ([[CABasicAnimation]])         |
| **Маски**              | Через `clip`                    | Через `mask`                         |
| **Градиенты**          | Линейные, радиальные            | Линейные, радиальные                 |

---

### Анимация градиента (с перерисовкой)

```swift
class AnimatedGradientView: UIView {
    private var gradientProgress: CGFloat = 0 {
        didSet {
            setNeedsDisplay()  // Перерисовываем при изменении
        }
    }
    
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else { return }
        
        let startColor = UIColor.red.cgColor
        let endColor = UIColor.blue.cgColor
        
        let colorSpace = CGColorSpaceCreateDeviceRGB()
        let locations: [CGFloat] = [0.0, gradientProgress]
        
        guard let gradient = CGGradient(colorsSpace: colorSpace,
                                        colors: [startColor, endColor] as CFArray,
                                        locations: locations) else { return }
        
        context.drawLinearGradient(gradient,
                                   start: CGPoint(x: rect.minX, y: rect.midY),
                                   end: CGPoint(x: rect.maxX, y: rect.midY),
                                   options: [])
    }
    
    func startAnimation(duration: TimeInterval) {
        let animation = CABasicAnimation(keyPath: "gradientProgress")
        animation.fromValue = 0.0
        animation.toValue = 1.0
        animation.duration = duration
        animation.repeatCount = .infinity
        animation.autoreverses = true
        
        self.layer.add(animation, forKey: "gradientAnimation")
    }
}
```

---

### Комбинирование с [[CAGradientLayer]]

```swift
class HybridGradientView: UIView {
    override class var layerClass: AnyClass {
        return CAGradientLayer.self
    }
    
    override func layoutSubviews() {
        super.layoutSubviews()
        
        guard let gradientLayer = layer as? CAGradientLayer else { return }
        
        gradientLayer.colors = [UIColor.red.cgColor, UIColor.blue.cgColor]
        gradientLayer.startPoint = CGPoint(x: 0, y: 0.5)
        gradientLayer.endPoint = CGPoint(x: 1, y: 0.5)
        
        // Добавляем дополнительный градиент через CGGradient
        DispatchQueue.main.async {
            self.setNeedsDisplay()
        }
    }
    
    override func draw(_ rect: CGRect) {
        super.draw(rect)
        
        guard let context = UIGraphicsGetCurrentContext() else { return }
        
        // Дополнительный градиент поверх CAGradientLayer
        let overlayGradient = CGGradient.linear(colors: [.white.withAlphaComponent(0.3), .clear])
        context.drawLinearGradient(overlayGradient!,
                                   start: CGPoint(x: rect.minX, y: rect.minY),
                                   end: CGPoint(x: rect.maxX, y: rect.maxY),
                                   options: [])
    }
}
```

---

### Эффект "сияния" (glow) с радиальным градиентом

```swift
class GlowEffectView: UIView {
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else { return }
        
        let colors = [UIColor.yellow.withAlphaComponent(0.8).cgColor,
                      UIColor.yellow.withAlphaComponent(0.0).cgColor]
        let colorSpace = CGColorSpaceCreateDeviceRGB()
        let locations: [CGFloat] = [0.0, 1.0]
        
        guard let gradient = CGGradient(colorsSpace: colorSpace,
                                        colors: colors as CFArray,
                                        locations: locations) else { return }
        
        let center = CGPoint(x: rect.midX, y: rect.midY)
        let radius = min(rect.width, rect.height) / 2
        
        context.drawRadialGradient(gradient,
                                   startCenter: center, startRadius: 0,
                                   endCenter: center, endRadius: radius,
                                   options: [])
    }
}
```

---

### Лучшие практики

1.  **Кэшируйте градиенты** (создавайте один раз, переиспользуйте).
2.  **Для статических градиентов используйте `CAGradientLayer`** — он проще и производительнее.
3.  **Для анимированных градиентов используйте `CAGradientLayer`** — Core Animation оптимизирован для этого.
4.  **Используйте `CGGradient` в `draw(_:)`** только для кастомного рисования, где нужен полный контроль.
5.  **Не создавайте градиенты в `draw(_:)` каждый раз** — выносите в `lazy var`.

```swift
lazy var cachedGradient: CGGradient? = {
    return CGGradient.linear(colors: [.red, .blue])
}()
```

---

### Итог

**`CGGradient`** — это мощный инструмент для создания градиентных заливок в Core Graphics. Он позволяет:

1.  **Создавать линейные и радиальные градиенты** с произвольным количеством цветов.
2.  **Рисовать градиенты в `CGContext`** с полным контролем над процессом.
3.  **Комбинировать с масками** для создания сложных форм.
4.  **Анимировать градиенты** (через перерисовку).
5.  **Интегрироваться с `CAGradientLayer`** для гибридных решений.

Понимание `CGGradient` необходимо для создания кастомных графических эффектов, сложных фонов и визуализации данных .