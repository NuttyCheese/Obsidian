**`CGPath`** — это структура (C-структура) из **Core Graphics** (Quartz), которая представляет **математический путь** (path) — последовательность линий, кривых Безье, дуг и прямоугольников.

Это **низкоуровневый** объект, который используется для:
- рисования произвольных форм в `CGContext`,
- создания масок,
- задания путей для `CAShapeLayer`,
- проверки попадания точки в область (`contains(_:)`),
- создания сложных clipping-путей и границ.

### Основные способы создания CGPath (2026 актуально)

| Способ создания                              | Пример кода                                                                 | Когда использовать |
|----------------------------------------------|-----------------------------------------------------------------------------|---------------------|
| Пустой путь                                  | `let path = CGMutablePath()`                                                | Начало построения пути |
| Прямоугольник                                | `CGPath(rect: rect, transform: nil)`                                        | Простые прямоугольники |
| Закруглённый прямоугольник                   | `CGPath(roundedRect: rect, cornerWidth: 12, cornerHeight: 12, transform: nil)` | Кнопки, карточки |
| Эллипс / круг                                | `CGPath(ellipseIn: rect, transform: nil)`                                   | Круги, овалы |
| Линия / ломаная                              | `path.move(to: start)`<br>`path.addLine(to: end)`                           | Полигоны, линии |
| Кривая Безье (квадратичная / кубическая)     | `path.addQuadCurve(to: end, control: control)`<br>`path.addCurve(to: end, control1: c1, control2: c2)` | Плавные кривые, анимации |
| Из `UIBezierPath`                            | `UIBezierPath(roundedRect: rect, cornerRadius: 12).cgPath`                  | Самый удобный и рекомендуемый способ |
| Из `CAShapeLayer`                            | `shapeLayer.path`                                                           | Когда работаете с CALayer |

### Самые популярные и рекомендуемые паттерны CGPath в 2026

#### 1. Самый частый способ — через UIBezierPath (рекомендуемый)

```swift
let path = UIBezierPath(roundedRect: CGRect(x: 20, y: 20, width: 200, height: 100),
                        cornerRadius: 16).cgPath

let shapeLayer = CAShapeLayer()
shapeLayer.path = path
shapeLayer.fillColor = UIColor.systemBlue.cgColor
shapeLayer.strokeColor = UIColor.black.cgColor
shapeLayer.lineWidth = 4
view.layer.addSublayer(shapeLayer)
```

#### 2. Ручное построение сложного пути (CGMutablePath)

```swift
let path = CGMutablePath()

// Начало в точке
path.move(to: CGPoint(x: 100, y: 100))

// Прямая линия
path.addLine(to: CGPoint(x: 200, y: 200))

// Квадратичная кривая Безье
path.addQuadCurve(to: CGPoint(x: 300, y: 100), control: CGPoint(x: 250, y: 300))

// Кубическая кривая Безье
path.addCurve(to: CGPoint(x: 400, y: 200),
              control1: CGPoint(x: 350, y: 50),
              control2: CGPoint(x: 450, y: 350))

// Закрыть путь (опционально)
path.closeSubpath()

// Применить к CAShapeLayer
let shape = CAShapeLayer()
shape.path = path
shape.strokeColor = UIColor.red.cgColor
shape.lineWidth = 3
shape.fillColor = nil
view.layer.addSublayer(shape)
```

#### 3. Проверка попадания точки в путь (очень полезно для hit-testing)

```swift
let path = UIBezierPath(ovalIn: CGRect(x: 50, y: 50, width: 200, height: 200)).cgPath

let point = CGPoint(x: 150, y: 150)
if path.contains(point) {
    print("Точка внутри эллипса")
}
```

#### 4. Создание маски для UIView

```swift
let maskPath = UIBezierPath(roundedRect: view.bounds,
                            byRoundingCorners: [.topLeft, .topRight],
                            cornerRadii: CGSize(width: 20, height: 20)).cgPath

let maskLayer = CAShapeLayer()
maskLayer.path = maskPath
view.layer.mask = maskLayer
```

### Лучшие практики CGPath в Swift 2026

- **Всегда** создавайте путь **через `UIBezierPath`** — это удобнее, читаемее и менее подвержено ошибкам  
- **Никогда** не создавайте `CGMutablePath` вручную, если можно обойтись `UIBezierPath`  
- **Для CAShapeLayer** — предпочтительнее `UIBezierPath.cgPath`  
- **Для сложных анимаций** — используйте `CABasicAnimation(keyPath: "path")` с `CGPath`  
- **Для hit-testing** — `CGPath.contains(_:)` — очень точный и быстрый  
- **Swift 6 strict concurrency** — `CGPath` полностью `Sendable`  
- **Документируйте** — пишите комментарий «CGPath — закруглённый прямоугольник для маски view (top corners 20 pt)»

**Короткий итог 2026**:
> `CGPath` — это **математический путь** для рисования, масок, аннотаций и hit-testing в Core Graphics.  
> В 2026 году:  
> - создавайте **всегда** через `UIBezierPath.cgPath`  
> - используйте для `CAShapeLayer.path`, `CGContext.addPath`, масок, проверки contains  
> - это **низкоуровневый**, но **очень мощный** инструмент для кастомных форм и UI  

Удачи с красивыми, точными и анимированными путями в твоём приложении! ✏️