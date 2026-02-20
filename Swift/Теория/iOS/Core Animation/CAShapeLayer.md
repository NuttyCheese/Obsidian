**`CAShapeLayer`** — это специальный подкласс **[[CALayer]]** из **QuartzCore** ([[Core Animation]]), который позволяет рисовать и анимировать **векторные пути** (paths) на экране.

Это один из самых мощных и часто используемых инструментов для создания кастомных UI-элементов в iOS/macOS-приложениях:
- закруглённые кнопки и карточки
- прогресс-бары и круговые индикаторы
- анимированные иконки и лоадеры
- маски сложной формы
- кастомные границы и подчёркивания
- графики, диаграммы, waveform-аудио

### Основные свойства CAShapeLayer (самые важные в 2026)

| Свойство                    | Тип                      | Что делает / зачем нужен                          | Самый частый сценарий         |
| --------------------------- | ------------------------ | ------------------------------------------------- | ----------------------------- |
| `path`                      | [[CGPath]]`?`            | Основной путь, который будет отрисован            | Всё рисование идёт через него |
| `fillColor`                 | [[CGColor]]`?`           | Цвет заливки ([[nil]] = без заливки)              | Фон фигуры                    |
| `strokeColor`               | `CGColor?`               | Цвет обводки (контура)                            | Границы, линии                |
| `lineWidth`                 | [[CGFloat]]              | Толщина линии обводки                             | 1–4 pt — стандарт             |
| `lineCap`                   | [[CAShapeLayerLineCap]]  | Стиль концов линии (butt, round, square)          | `.round` — для плавности      |
| `lineJoin`                  | [[CAShapeLayerLineJoin]] | Стиль соединения линий (miter, round, bevel)      | `.round` — чаще всего         |
| `strokeStart` / `strokeEnd` | `CGFloat` (0…1)          | Анимация обводки (от 0 до 1)                      | Анимированный stroke          |
| `fillRule`                  | [[CAShapeLayerFillRule]] | Правило заливки сложных путей (even-odd, nonzero) | `.evenOdd` — для дыр          |
| `miterLimit`                | `CGFloat`                | Ограничение длины углов при `lineJoin = .miter`   | Обычно 10–20                  |

### Самые популярные и рекомендуемые паттерны CAShapeLayer в 2026

#### 1. Закруглённая карточка / кнопка (самый частый случай)

```swift
let cardLayer = CAShapeLayer()
let path = UIBezierPath(roundedRect: bounds,
                        cornerRadius: 16).cgPath
cardLayer.path = path
cardLayer.fillColor = UIColor.systemBackground.cgColor
cardLayer.strokeColor = UIColor.systemGray4.cgColor
cardLayer.lineWidth = 1
cardLayer.shadowColor = UIColor.black.cgColor
cardLayer.shadowOpacity = 0.1
cardLayer.shadowRadius = 8
cardLayer.shadowOffset = CGSize(width: 0, height: 4)

layer.addSublayer(cardLayer)
```

#### 2. Круговой прогресс-бар (очень популярный анимированный индикатор)

```swift
class CircularProgressView: UIView {
    private let shapeLayer = CAShapeLayer()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        
        let center = CGPoint(x: bounds.midX, y: bounds.midY)
        let radius = min(bounds.width, bounds.height) / 2 - 10
        
        let path = UIBezierPath(arcCenter: center,
                               radius: radius,
                               startAngle: -CGFloat.pi / 2,
                               endAngle: 3 * CGFloat.pi / 2,
                               clockwise: true).cgPath
        
        shapeLayer.path = path
        shapeLayer.fillColor = nil
        shapeLayer.strokeColor = UIColor.systemBlue.cgColor
        shapeLayer.lineWidth = 12
        shapeLayer.lineCap = .round
        shapeLayer.strokeEnd = 0  // от 0 до 1
        
        layer.addSublayer(shapeLayer)
    }
    
    func setProgress(_ progress: CGFloat, animated: Bool = true) {
        let animation = CABasicAnimation(keyPath: "strokeEnd")
        animation.fromValue = shapeLayer.strokeEnd
        animation.toValue = progress
        animation.duration = animated ? 0.6 : 0
        animation.timingFunction = CAMediaTimingFunction(name: .easeOut)
        
        shapeLayer.strokeEnd = progress
        shapeLayer.add(animation, forKey: "progress")
    }
}
```

#### 3. Маска сложной формы для [[UIView]]

```swift
let maskPath = UIBezierPath()
maskPath.move(to: CGPoint(x: 0, y: 0))
maskPath.addLine(to: CGPoint(x: bounds.width, y: 0))
maskPath.addLine(to: CGPoint(x: bounds.width, y: bounds.height - 40))
maskPath.addQuadCurve(to: CGPoint(x: bounds.width / 2, y: bounds.height),
                      control: CGPoint(x: bounds.width, y: bounds.height))
maskPath.addQuadCurve(to: CGPoint(x: 0, y: bounds.height - 40),
                      control: CGPoint(x: 0, y: bounds.height))
maskPath.close()

let maskLayer = CAShapeLayer()
maskLayer.path = maskPath.cgPath
view.layer.mask = maskLayer
```

### Лучшие практики CAShapeLayer в Swift 2026

- **Всегда** создавайте путь через **`UIBezierPath`** → `.cgPath` — это читаемо и безопасно  
- **Не добавляйте** `CAShapeLayer` в `draw(_:)` — лучше один раз в `init` или `layoutSubviews`  
- **Для анимаций** — используйте `CABasicAnimation(keyPath: "strokeEnd")`, `"path"`, `"transform"`  
- **Для производительности** — минимизируйте количество `CAShapeLayer` на экране (лучше один сложный путь)  
- **Для SwiftUI** — предпочитайте `Path` и `Shape` — они построены поверх `CAShapeLayer`  
- **Swift 6 strict concurrency** — `CAShapeLayer` — только на главном потоке (`@MainActor`)  
- **Документируйте** — пишите комментарий «CAShapeLayer — анимированный круговой прогресс-бар (strokeEnd от 0 до 1)»

**Короткий итог 2026**:
> `CAShapeLayer` — это **векторный слой** для рисования и анимации произвольных форм через `CGPath`.  
> В 2026 году:  
> - создавайте путь **через `UIBezierPath.cgPath`**  
> - используйте для: прогресс-баров, масок, кастомных кнопок, линий, иконок  
> - анимируйте `strokeEnd`, `path`, `transform`  
> - это **самый мощный** и **самый гибкий** инструмент для кастомного векторного UI в [[UIKit]]  
