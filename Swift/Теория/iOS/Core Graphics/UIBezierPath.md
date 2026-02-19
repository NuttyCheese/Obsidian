**`UIBezierPath`** — это высокоуровневая обёртка над [[CGPath]] в [[UIKit]], которая предоставляет **очень удобный и безопасный** способ создания и управления **векторными путями** в [[iOS]]-приложениях.

Это **самый рекомендуемый** инструмент для большинства задач с формами, масками, анимациями и кастомным рисованием в [[UIKit]] (и часто используется в [[SwiftUI]] под капотом).

### Почему UIBezierPath — это лучший выбор в 2026 году

| Преимущество                       | Почему это важно в 2026                                       | По сравнению с чистым [[CGPath]] |
| ---------------------------------- | ------------------------------------------------------------- | -------------------------------- |
| **Читаемый и declarative [[API]]** | `addLine(to:)`, `addQuadCurve(to:control:)`, `addArc(...)`    | Значительно проще и понятнее     |
| **Безопасность**                   | Не нужно вручную управлять `CGMutablePath` и `closeSubpath()` | Меньше ошибок и утечек           |
| **Поддержка [[SwiftUI]]**          | `Path` в SwiftUI построен поверх `UIBezierPath`               | Легко переносить код             |
| **Анимация**                       | Легко анимировать `path` в `CAShapeLayer`                     | Стандарт для stroke-анимаций     |
| **Маски и clipping**               | Прямо преобразуется в `CGPath` для `layer.mask`               | Самый частый сценарий            |
| **Производительность**             | Никакой разницы с `CGPath` (это просто обёртка)               | Полная совместимость             |

### Основные методы создания UIBezierPath

| Метод / Конструктор                              | Что создаёт                                                  | Самый частый сценарий |
|--------------------------------------------------|--------------------------------------------------------------|-----------------------|
| `init(rect:)`                                    | Прямоугольник                                                | Фон, карточки         |
| `init(roundedRect:cornerRadius:)`                | Закруглённый прямоугольник                                   | Кнопки, карточки      |
| `init(roundedRect:byRoundingCorners:cornerRadii:)` | Частично закруглённый прямоугольник                         | Bottom sheet, модалки  |
| `init(ovalIn:)`                                  | Эллипс / круг внутри прямоугольника                          | Круглые аватарки      |
| `init(arcCenter:radius:startAngle:endAngle:clockwise:)` | Дуга / сектор / полный круг                                | Прогресс-бары         |
| `init()` + методы `move(to:)`, `addLine(to:)`, `addQuadCurve`, `addCurve` | Произвольный путь                                            | Кастомные формы       |

### Самые популярные и рекомендуемые паттерны UIBezierPath в 2026

#### 1. Закруглённая карточка / кнопка (самый частый случай)

```swift
let path = UIBezierPath(roundedRect: bounds,
                        cornerRadius: 16)

let shapeLayer = CAShapeLayer()
shapeLayer.path = path.cgPath
shapeLayer.fillColor = UIColor.systemBackground.cgColor
shapeLayer.strokeColor = UIColor.systemGray4.cgColor
shapeLayer.lineWidth = 1
shapeLayer.shadowColor = UIColor.black.cgColor
shapeLayer.shadowOpacity = 0.12
shapeLayer.shadowRadius = 8
shapeLayer.shadowOffset = CGSize(width: 0, height: 4)

layer.addSublayer(shapeLayer)
```

#### 2. Круговой прогресс-бар / индикатор загрузки (очень популярный)

```swift
class CircularProgressView: UIView {
    private let progressLayer = CAShapeLayer()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        
        let center = CGPoint(x: bounds.midX, y: bounds.midY)
        let radius = min(bounds.width, bounds.height) / 2 - 10
        
        let path = UIBezierPath(arcCenter: center,
                               radius: radius,
                               startAngle: -CGFloat.pi / 2,
                               endAngle: CGFloat.pi * 2 - CGFloat.pi / 2,
                               clockwise: true)
        
        progressLayer.path = path.cgPath
        progressLayer.fillColor = nil
        progressLayer.strokeColor = UIColor.systemBlue.cgColor
        progressLayer.lineWidth = 12
        progressLayer.lineCap = .round
        progressLayer.strokeEnd = 0
        
        layer.addSublayer(progressLayer)
    }
    
    func setProgress(_ progress: CGFloat, animated: Bool = true) {
        let animation = CABasicAnimation(keyPath: "strokeEnd")
        animation.fromValue = progressLayer.strokeEnd
        animation.toValue = progress
        animation.duration = animated ? 0.6 : 0
        animation.timingFunction = CAMediaTimingFunction(name: .easeOut)
        
        progressLayer.strokeEnd = progress
        progressLayer.add(animation, forKey: "progress")
    }
}
```

#### 3. Частично закруглённые углы (bottom sheet, модальные окна)

```swift
let path = UIBezierPath(roundedRect: bounds,
                        byRoundingCorners: [.topLeft, .topRight],
                        cornerRadii: CGSize(width: 24, height: 24))

let maskLayer = CAShapeLayer()
maskLayer.path = path.cgPath
view.layer.mask = maskLayer
```

#### 4. Кастомная форма + анимация stroke

```swift
let path = UIBezierPath()
path.move(to: CGPoint(x: 50, y: 50))
path.addLine(to: CGPoint(x: 200, y: 200))
path.addQuadCurve(to: CGPoint(x: 300, y: 100), controlPoint: CGPoint(x: 250, y: 300))
path.addCurve(to: CGPoint(x: 400, y: 200),
              controlPoint1: CGPoint(x: 350, y: 50),
              controlPoint2: CGPoint(x: 450, y: 350))
path.close()

let shapeLayer = CAShapeLayer()
shapeLayer.path = path.cgPath
shapeLayer.strokeColor = UIColor.systemPurple.cgColor
shapeLayer.lineWidth = 6
shapeLayer.fillColor = nil
shapeLayer.lineCap = .round
shapeLayer.strokeEnd = 0

layer.addSublayer(shapeLayer)

// Анимация обводки
let animation = CABasicAnimation(keyPath: "strokeEnd")
animation.fromValue = 0
animation.toValue = 1
animation.duration = 2
animation.timingFunction = CAMediaTimingFunction(name: .easeInEaseOut)
shapeLayer.add(animation, forKey: "stroke")
shapeLayer.strokeEnd = 1
```

### Лучшие практики UIBezierPath + [[CAShapeLayer]] в Swift 2026

- **Всегда** создавайте путь через `UIBezierPath` — это **самый читаемый** и **самый безопасный** способ  
- **Не добавляйте** `CAShapeLayer` в `draw(_:)` — лучше один раз в `init` / `layoutSubviews`  
- **Для анимаций** — анимируйте `strokeEnd`, `path`, `transform.scale`, `transform.rotation`  
- **Для производительности** — минимизируйте количество `CAShapeLayer` (лучше один сложный путь)  
- **Для SwiftUI** — используйте `Path` и `Shape` — они построены поверх `UIBezierPath` / `CGPath`  
- **Swift 6 strict concurrency** — `UIBezierPath` и `CAShapeLayer` — только на главном потоке ([[@MainActor]])  
- **Документируйте** — пишите комментарий «UIBezierPath — закруглённый путь для маски view (top corners 24 pt)»

**Короткий итог 2026**:
> `UIBezierPath` — это **самый удобный** способ создавать векторные пути для `CAShapeLayer`, масок, drawing и анимаций.  
> В 2026 году:  
> - создавайте **всегда** через `UIBezierPath` (не чистый `CGMutablePath`)  
> - используйте для: прогресс-баров, масок, кастомных кнопок, линий, waveform  
> - анимируйте `strokeEnd` и `path` в `CAShapeLayer`  
> - это **самый популярный** и **самый выразительный** инструмент векторной графики в UIKit  
