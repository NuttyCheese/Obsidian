#core-animation #cagradientlayer #gradient #animation #ios #swift #uikit

---

## CAGradientLayer — Градиентные заливки в Core Animation

### Определение

**`CAGradientLayer`** — это подкласс [[CALayer]] в [[Core Animation]], который предоставляет **градиентную заливку** — плавный переход между двумя или более цветами вдоль заданного направления. В отличие от [[CGGradient]] (который используется для низкоуровневого рисования в [[CGContext]]), `CAGradientLayer` является самостоятельным слоем, который можно добавлять в иерархию слоёв, анимировать и комбинировать с другими слоями.

### Зачем это знать iOS-разработчику?

1.  **Фоновые заливки:** Создание красивых градиентных фонов для `UIView` без необходимости переопределять `draw(_:)`.
2.  **Анимация:** Плавное изменение цветов, позиций и направлений градиента.
3.  **Маски:** Комбинирование градиентов с масками для создания сложных эффектов.
4.  **Кнопки и элементы интерфейса:** Создание "живых" кнопок с градиентными заливками.
5.  **Производительность:** Отрисовка на [[GPU]], что делает анимации очень плавными.

---

### Типы градиентов

| Тип | Свойство | Описание |
|---|---|---|
| **Линейный (Linear)** | `type = .axial` (по умолчанию) | Цвета изменяются вдоль прямой линии между двумя точками. |
| **Радиальный (Radial)** | `type = .radial` | Цвета изменяются от центральной точки к внешнему радиусу. |
| **Коноидальный (Conical)** | `type = .conic` | Цвета изменяются вокруг центральной точки (как в цветовом круге). |

---

### Создание CAGradientLayer

#### 1. **Базовый линейный градиент**

```swift
import UIKit

class GradientViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let gradientLayer = CAGradientLayer()
        gradientLayer.frame = view.bounds
        gradientLayer.colors = [UIColor.red.cgColor, UIColor.blue.cgColor]
        gradientLayer.startPoint = CGPoint(x: 0, y: 0.5)
        gradientLayer.endPoint = CGPoint(x: 1, y: 0.5)
        
        view.layer.addSublayer(gradientLayer)
    }
}
```

#### 2. **Вертикальный градиент**

```swift
let gradientLayer = CAGradientLayer()
gradientLayer.frame = view.bounds
gradientLayer.colors = [UIColor.systemTeal.cgColor, UIColor.systemPurple.cgColor]
gradientLayer.startPoint = CGPoint(x: 0.5, y: 0)
gradientLayer.endPoint = CGPoint(x: 0.5, y: 1)

view.layer.addSublayer(gradientLayer)
```

#### 3. **Диагональный градиент**

```swift
gradientLayer.startPoint = CGPoint(x: 0, y: 0)
gradientLayer.endPoint = CGPoint(x: 1, y: 1)
```

---

### Ключевые свойства

| Свойство          | Тип                     | Описание                                                                        |
| ----------------- | ----------------------- | ------------------------------------------------------------------------------- |
| **`colors`**      | `[CGColor]?`            | Массив цветов градиента.                                                        |
| **`locations`**   | `[NSNumber]?`           | Позиции цветов (от 0.0 до 1.0). Если `nil`, цвета распределены равномерно.      |
| **`startPoint`**  | [[CGPoint]]             | Начальная точка градиента (в единицах координат слоя). По умолчанию `(0.5, 0)`. |
| **`endPoint`**    | `CGPoint`               | Конечная точка градиента. По умолчанию `(0.5, 1)`.                              |
| **`type`**        | [[CAGradientLayerType]] | Тип градиента: `.axial`, `.radial`, `.conic`.                                   |
| **`center`**      | `CGPoint`               | Центр для радиального/коноидального градиента.                                  |
| **`radius`**      | [[CGFloat]]             | Радиус для радиального градиента.                                               |
| **`startRadius`** | `CGFloat`               | Начальный радиус для радиального градиента.                                     |
| **`endRadius`**   | `CGFloat`               | Конечный радиус для радиального градиента.                                      |

---

### Работа с locations

#### 1. **Равномерное распределение (без locations)**

```swift
gradientLayer.colors = [UIColor.red.cgColor, UIColor.green.cgColor, UIColor.blue.cgColor]
// Цвета распределены равномерно: 0%, 50%, 100%
```

#### 2. **Кастомное распределение**

```swift
gradientLayer.colors = [UIColor.red.cgColor, UIColor.yellow.cgColor, UIColor.green.cgColor]
gradientLayer.locations = [0.0, 0.3, 1.0]
// Красный: 0%, Жёлтый: 30%, Зелёный: 100%
```

#### 3. **Остановка в одной точке (резкий переход)**

```swift
gradientLayer.colors = [UIColor.red.cgColor, UIColor.blue.cgColor]
gradientLayer.locations = [0.0, 0.5]
// Резкий переход на 50%
```

---

### Примеры градиентов

#### 1. **Многоцветный градиент (радуга)**

```swift
class RainbowGradientView: UIView {
    override class var layerClass: AnyClass {
        return CAGradientLayer.self
    }
    
    override func layoutSubviews() {
        super.layoutSubviews()
        
        guard let gradientLayer = layer as? CAGradientLayer else { return }
        
        gradientLayer.colors = [
            UIColor.red.cgColor,
            UIColor.orange.cgColor,
            UIColor.yellow.cgColor,
            UIColor.green.cgColor,
            UIColor.blue.cgColor,
            UIColor.purple.cgColor
        ]
        gradientLayer.startPoint = CGPoint(x: 0, y: 0)
        gradientLayer.endPoint = CGPoint(x: 1, y: 1)
    }
}
```

#### 2. **Радиальный градиент**

```swift
let radialGradient = CAGradientLayer()
radialGradient.frame = view.bounds
radialGradient.type = .radial
radialGradient.colors = [UIColor.yellow.cgColor, UIColor.orange.cgColor, UIColor.red.cgColor]
radialGradient.center = CGPoint(x: 0.5, y: 0.5)
radialGradient.startRadius = 0
radialGradient.endRadius = view.bounds.width / 2

view.layer.addSublayer(radialGradient)
```

#### 3. **Коноидальный градиент (цветовой круг)**

```swift
let conicGradient = CAGradientLayer()
conicGradient.frame = view.bounds
conicGradient.type = .conic
conicGradient.colors = [
    UIColor.red.cgColor,
    UIColor.yellow.cgColor,
    UIColor.green.cgColor,
    UIColor.blue.cgColor,
    UIColor.purple.cgColor,
    UIColor.red.cgColor
]
conicGradient.center = CGPoint(x: 0.5, y: 0.5)

view.layer.addSublayer(conicGradient)
```

#### 4. **Градиент с прозрачностью**

```swift
let transparentGradient = CAGradientLayer()
transparentGradient.frame = view.bounds
transparentGradient.colors = [
    UIColor.black.withAlphaComponent(0.8).cgColor,
    UIColor.black.withAlphaComponent(0.0).cgColor
]
transparentGradient.startPoint = CGPoint(x: 0.5, y: 0)
transparentGradient.endPoint = CGPoint(x: 0.5, y: 1)

view.layer.addSublayer(transparentGradient)
```

---

### Анимация градиента

#### 1. **Анимация цветов**

```swift
class AnimatedGradientView: UIView {
    private let gradientLayer = CAGradientLayer()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupGradient()
        animateGradient()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        setupGradient()
        animateGradient()
    }
    
    private func setupGradient() {
        gradientLayer.frame = bounds
        gradientLayer.colors = [UIColor.red.cgColor, UIColor.blue.cgColor]
        layer.addSublayer(gradientLayer)
    }
    
    private func animateGradient() {
        let animation = CABasicAnimation(keyPath: "colors")
        animation.fromValue = [UIColor.red.cgColor, UIColor.blue.cgColor]
        animation.toValue = [UIColor.green.cgColor, UIColor.yellow.cgColor]
        animation.duration = 2.0
        animation.autoreverses = true
        animation.repeatCount = .infinity
        
        gradientLayer.add(animation, forKey: "colorChange")
    }
    
    override func layoutSubviews() {
        super.layoutSubviews()
        gradientLayer.frame = bounds
    }
}
```

#### 2. **Анимация startPoint и endPoint**

```swift
let moveAnimation = CABasicAnimation(keyPath: "startPoint")
moveAnimation.fromValue = CGPoint(x: 0, y: 0)
moveAnimation.toValue = CGPoint(x: 1, y: 1)
moveAnimation.duration = 3.0
moveAnimation.autoreverses = true
moveAnimation.repeatCount = .infinity

gradientLayer.add(moveAnimation, forKey: "moveStartPoint")
```

#### 3. **Анимация locations**

```swift
let locationsAnimation = CABasicAnimation(keyPath: "locations")
locationsAnimation.fromValue = [0.0, 0.5, 1.0]
locationsAnimation.toValue = [0.0, 0.8, 1.0]
locationsAnimation.duration = 1.5
locationsAnimation.autoreverses = true
locationsAnimation.repeatCount = .infinity

gradientLayer.add(locationsAnimation, forKey: "locationsChange")
```

---

### Комбинирование с масками

```swift
class MaskedGradientView: UIView {
    override func layoutSubviews() {
        super.layoutSubviews()
        
        // Градиентный слой
        let gradientLayer = CAGradientLayer()
        gradientLayer.frame = bounds
        gradientLayer.colors = [UIColor.red.cgColor, UIColor.blue.cgColor]
        
        // Маска (круг)
        let maskLayer = CAShapeLayer()
        let path = UIBezierPath(ovalIn: CGRect(x: 50, y: 50, width: 200, height: 200))
        maskLayer.path = path.cgPath
        
        gradientLayer.mask = maskLayer
        layer.addSublayer(gradientLayer)
    }
}
```

---

### CAGradientLayer в [[UIView]] (обёртка)

```swift
class GradientView: UIView {
    var colors: [UIColor] = [.red, .blue] {
        didSet {
            updateGradient()
        }
    }
    
    var startPoint: CGPoint = CGPoint(x: 0, y: 0.5) {
        didSet {
            updateGradient()
        }
    }
    
    var endPoint: CGPoint = CGPoint(x: 1, y: 0.5) {
        didSet {
            updateGradient()
        }
    }
    
    private var gradientLayer: CAGradientLayer?
    
    override class var layerClass: AnyClass {
        return CAGradientLayer.self
    }
    
    private func updateGradient() {
        guard let gradientLayer = layer as? CAGradientLayer else { return }
        gradientLayer.colors = colors.map { $0.cgColor }
        gradientLayer.startPoint = startPoint
        gradientLayer.endPoint = endPoint
    }
    
    override func layoutSubviews() {
        super.layoutSubviews()
        updateGradient()
    }
}

// Использование
let gradientView = GradientView(frame: CGRect(x: 0, y: 0, width: 300, height: 100))
gradientView.colors = [.systemTeal, .systemPurple]
gradientView.startPoint = CGPoint(x: 0, y: 0)
gradientView.endPoint = CGPoint(x: 1, y: 1)
```

---

### CAGradientLayer vs [[CGGradient]]

| Характеристика         | `CAGradientLayer`                  | `CGGradient`            |
| ---------------------- | ---------------------------------- | ----------------------- |
| **Уровень**            | [[Core Animation]]                 | [[Core Graphics]]       |
| **Использование**      | Добавление как `sublayer`          | Рисование в `draw(_:)`  |
| **Анимация**           | ✅ (на GPU)                         | ❌ (требует перерисовки) |
| **Производительность** | Высокая ([[GPU]])                  | Средняя ([[CPU]])       |
| **Типы градиентов**    | Линейный, радиальный, коноидальный | Линейный, радиальный    |
| **Маски**              | Через `mask`                       | Через `clip`            |

---

### Работа с несколькими слоями

```swift
class MultiGradientView: UIView {
    override func layoutSubviews() {
        super.layoutSubviews()
        
        // Первый градиент (фон)
        let backgroundGradient = CAGradientLayer()
        backgroundGradient.frame = bounds
        backgroundGradient.colors = [UIColor.darkGray.cgColor, UIColor.black.cgColor]
        layer.addSublayer(backgroundGradient)
        
        // Второй градиент (эффект свечения)
        let glowGradient = CAGradientLayer()
        glowGradient.frame = bounds
        glowGradient.colors = [UIColor.yellow.withAlphaComponent(0.3).cgColor,
                               UIColor.clear.cgColor]
        glowGradient.startPoint = CGPoint(x: 0.5, y: 0)
        glowGradient.endPoint = CGPoint(x: 0.5, y: 1)
        layer.addSublayer(glowGradient)
    }
}
```

---

### Лучшие практики

1.  **Всегда обновляйте `frame` в `layoutSubviews`** при использовании в `UIView`.
2.  **Используйте `class var layerClass` для создания `UIView` с градиентом по умолчанию.**
3.  **Кэшируйте градиентный слой**, если он не меняется.
4.  **Для анимаций используйте `CABasicAnimation`**, а не ручную перерисовку.
5.  **Помните, что `colors` принимает `CGColor`, а не `UIColor`.**

```swift
// Правильно
gradientLayer.colors = [UIColor.red.cgColor, UIColor.blue.cgColor]

// Неправильно
gradientLayer.colors = [UIColor.red, UIColor.blue]  // Ошибка компиляции
```

---

### Итог

**`CAGradientLayer`** — это мощный и гибкий инструмент для создания градиентных заливок в Core Animation. Он позволяет:

1.  **Создавать линейные, радиальные и коноидальные градиенты.**
2.  **Анимировать цвета, позиции и направления** с высокой производительностью.
3.  **Комбинировать с масками** для создания сложных форм.
4.  **Использовать как фоновый слой** для `UIView` без переопределения `draw(_:)`.
5.  **Создавать многослойные градиентные эффекты.**

Понимание `CAGradientLayer` необходимо для создания современных, анимированных интерфейсов с красивыми градиентными эффектами .