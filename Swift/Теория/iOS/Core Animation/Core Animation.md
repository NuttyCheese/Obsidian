**Core Animation** (CA) — это мощный и высокопроизводительный фреймворк Apple для создания **анимаций** и управления **слоями** (layers) в [[iOS]], iPadOS, macOS, watchOS и tvOS.

Он лежит в основе всей современной анимации в [[UIKit]] и [[SwiftUI]] (под капотом SwiftUI использует именно Core Animation).

Core Animation работает **не на [[CPU]]**, а на **GPU** → анимации очень плавные (60/120 fps), даже при большом количестве элементов.

### Основные концепции Core Animation (2026 актуально)

| Концепция                     | Класс / Протокол             | Что это такое                                      | Самый частый сценарий |
|-------------------------------|------------------------------|----------------------------------------------------|------------------------|
| **Слой**                      | `CALayer`                    | Базовый объект для всего, что видно на экране      | Фон, градиенты, тени, маски |
| **Shape Layer**               | `CAShapeLayer`               | Векторный слой с путём (`CGPath` / `UIBezierPath`) | Прогресс-бары, иконки, маски, анимированные линии |
| **Анимация**                  | `CAAnimation` (базовый)      | Абстрактный класс всех анимаций                    | — |
| **Basic Animation**           | `CABasicAnimation`           | Простая анимация от значения A до значения B       | Самая частая анимация |
| **KeyFrame Animation**        | `CAKeyframeAnimation`        | Анимация по ключевым кадрам (keyframes)            | Сложные траектории, bounce |
| **Spring Animation**          | `CASpringAnimation`          | Пружинная анимация (физически реалистичная)        | Bounce, overshoot |
| **Animation Group**           | `CAAnimationGroup`           | Группа анимаций, выполняемых одновременно          | Комплексные эффекты |
| **Implicit Animation**        | —                            | Автоматическая анимация при изменении свойств слоя | `layer.opacity = 0` → плавное затухание |
| **Transaction**               | `CATransaction`              | Управление группой изменений (отключение анимации) | `CATransaction.disableActions()` |

### Самые популярные и рекомендуемые паттерны Core Animation в 2026

#### 1. Простая анимация свойства (самый частый)

```swift
let layer = CAShapeLayer()
layer.path = UIBezierPath(ovalIn: CGRect(x: 50, y: 50, width: 100, height: 100)).cgPath
layer.fillColor = UIColor.systemBlue.cgColor
view.layer.addSublayer(layer)

// Анимация изменения цвета
let animation = CABasicAnimation(keyPath: "fillColor")
animation.fromValue = UIColor.systemBlue.cgColor
animation.toValue   = UIColor.systemPurple.cgColor
animation.duration  = 1.5
animation.autoreverses = true
animation.repeatCount = .greatestFiniteMagnitude

layer.add(animation, forKey: "colorPulse")
```

#### 2. Анимация stroke (прогресс, загрузка, drawing)

```swift
let path = UIBezierPath(arcCenter: CGPoint(x: 100, y: 100),
                        radius: 80,
                        startAngle: -CGFloat.pi / 2,
                        endAngle: CGFloat.pi * 3 / 2,
                        clockwise: true).cgPath

let shapeLayer = CAShapeLayer()
shapeLayer.path = path
shapeLayer.strokeColor = UIColor.systemGreen.cgColor
shapeLayer.lineWidth = 12
shapeLayer.lineCap = .round
shapeLayer.fillColor = nil
shapeLayer.strokeEnd = 0

view.layer.addSublayer(shapeLayer)

// Анимация заполнения
let animation = CABasicAnimation(keyPath: "strokeEnd")
animation.fromValue = 0
animation.toValue   = 1
animation.duration  = 2
animation.timingFunction = CAMediaTimingFunction(name: .easeInEaseOut)

shapeLayer.add(animation, forKey: "progress")
shapeLayer.strokeEnd = 1
```

#### 3. Отключение анимации (очень важно для производительности)

```swift
CATransaction.begin()
CATransaction.setDisableActions(true)  // ← отключает все неявные анимации

layer.opacity = 0
layer.position = CGPoint(x: 200, y: 200)

CATransaction.commit()
```

#### 4. Группа анимаций (одновременно несколько эффектов)

```swift
let group = CAAnimationGroup()
group.duration = 1.2
group.animations = [
    // Масштаб
    {
        let anim = CABasicAnimation(keyPath: "transform.scale")
        anim.fromValue = 0.5
        anim.toValue = 1.0
        return anim
    }(),
    // Прозрачность
    {
        let anim = CABasicAnimation(keyPath: "opacity")
        anim.fromValue = 0
        anim.toValue = 1
        return anim
    }()
]

layer.add(group, forKey: "appear")
```

### Лучшие практики Core Animation в Swift 2026

- **Всегда** работайте с Core Animation **на главном потоке** (`@MainActor`)  
- **Предпочитайте** `CABasicAnimation` и `CASpringAnimation` — они самые простые и производительные  
- **Для сложных траекторий** — используйте `CAKeyframeAnimation` с `path`  
- **Отключайте** неявные анимации через `CATransaction.setDisableActions(true)` при массовых изменениях  
- **Для SwiftUI** — почти все анимации делаются через `.animation()`, `.transition()`, `.matchedGeometryEffect` — Core Animation под капотом  
- **Для Metal / SceneKit** — используйте `CAMetalLayer` вместо обычных слоёв  
- **Документируйте** — пишите комментарий «CABasicAnimation — анимация strokeEnd для кругового прогресс-бара»

**Короткий итог 2026**:
> Core Animation — это **GPU-ускоренный движок анимаций и слоёв**, на котором построен весь современный UI в iOS.  
> В 2026 году:  
> - основной слой — `CALayer` / `CAShapeLayer`  
> - основные анимации — `CABasicAnimation`, `CASpringAnimation`, `CAAnimationGroup`  
> - пути — через `UIBezierPath` → `.cgPath`  
> - отключайте анимации при массовых изменениях через `CATransaction`  
> Это **самый мощный** инструмент для плавных, производительных и сложных анимаций в UIKit.

Удачи с невероятно плавными и современными анимациями в твоём приложении! ✨🚀