**`CGLineCap`** — это перечисление (enum) в **Core Graphics**, которое определяет **стиль концов линий** (line cap style) при рисовании в контексте (`CGContext`) или использовании в `CAShapeLayer`.

Оно применяется к свойству `lineCap` и управляет тем, как выглядят **концы** открытой линии или незакрытого пути.

### Все возможные значения CGLineCap

| Значение в Swift                        | Константа в C / Objective-C      | Как выглядит конец линии                              | Когда использовать (самые частые случаи) |
|-----------------------------------------|----------------------------------|-------------------------------------------------------|-------------------------------------------|
| `.butt`                                 | `kCGLineCapButt`                 | Плоский срез ровно по конечной точке                 | По умолчанию — самый резкий и аккуратный  |
| `.round`                                | `kCGLineCapRound`                | Полукруглый (радиус = половине толщины линии)         | Самый популярный — мягкие, приятные концы |
| `.square`                               | `kCGLineCapSquare`               | Квадратный выступ (длина = половине толщины линии)    | Редко — когда нужен чёткий квадратный конец |

### Визуальное сравнение (на примере линии толщиной 20 pt)

Представьте горизонтальную линию:

```swift
let path = CGMutablePath()
path.move(to: CGPoint(x: 50, y: 100))
path.addLine(to: CGPoint(x: 300, y: 100))

let context = UIGraphicsGetCurrentContext()!
context.addPath(path)
context.setLineWidth(20)
context.setStrokeColor(UIColor.systemBlue.cgColor)
```

Теперь меняем только `lineCap`:

| CGLineCap    | Визуальный результат (концы линии)                     | Описание |
|--------------|--------------------------------------------------------|----------|
| `.butt`      | Резкий плоский срез                                    | Самый строгий, точный, без выступов |
| `.round`     | Полукруглые "шапочки" на концах                        | Самый мягкий и часто используемый |
| `.square`    | Квадратные выступы (как маленькие прямоугольники)      | Специфический "квадратный" конец |

### Как использовать CGLineCap в коде (самые частые паттерны 2026)

#### 1. В CGContext (рисование в UIView)

```swift
override func draw(_ rect: CGRect) {
    guard let context = UIGraphicsGetCurrentContext() else { return }
    
    let path = CGMutablePath()
    path.move(to: CGPoint(x: 50, y: 100))
    path.addLine(to: CGPoint(x: 300, y: 100))
    
    context.addPath(path)
    context.setLineWidth(20)
    context.setStrokeColor(UIColor.systemPurple.cgColor)
    context.setLineCap(.round)          // ← Самый популярный выбор
    context.strokePath()
}
```

#### 2. В CAShapeLayer (самый частый сценарий в UIKit)

```swift
let path = UIBezierPath(arcCenter: CGPoint(x: 100, y: 100),
                        radius: 80,
                        startAngle: -CGFloat.pi / 2,
                        endAngle: CGFloat.pi * 1.5,
                        clockwise: true).cgPath

let shapeLayer = CAShapeLayer()
shapeLayer.path = path
shapeLayer.fillColor = nil
shapeLayer.strokeColor = UIColor.systemGreen.cgColor
shapeLayer.lineWidth = 16
shapeLayer.lineCap = .round          // ← для плавных концов прогресс-бара
shapeLayer.strokeEnd = 0

view.layer.addSublayer(shapeLayer)
```

#### 3. Сравнение всех трёх стилей в одном примере

```swift
let yPositions: [CGFloat] = [80, 140, 200]
let caps: [CGLineCap] = [.butt, .round, .square]
let labels = ["butt", "round", "square"]

for (y, (cap, label)) in zip(yPositions, zip(caps, labels)) {
    let path = CGMutablePath()
    path.move(to: CGPoint(x: 50, y: y))
    path.addLine(to: CGPoint(x: 300, y: y))
    
    context.addPath(path)
    context.setLineWidth(20)
    context.setStrokeColor(UIColor.systemBlue.cgColor)
    context.setLineCap(cap)
    context.strokePath()
    
    // Подпись
    let text = NSAttributedString(string: label, attributes: [.foregroundColor: UIColor.black, .font: UIFont.systemFont(ofSize: 14)])
    text.draw(at: CGPoint(x: 320, y: y - 7))
}
```

### Рекомендации и лучшие практики (2026)

- **По умолчанию** — оставляйте `.butt` (самый точный и геометрически чистый)  
- **Для большинства UI-элементов** — переключайтесь на `.round` — выглядит мягче, современнее и приятнее  
- **Для `.square`** — используйте только осознанно (очень редкие стилизованные случаи, например, пиксель-арт)  
- **Всегда комбинируйте** с `lineJoin = .round` — вместе дают самый плавный результат  
- **Для анимаций stroke** — `.round` почти всегда лучше (концы не обрезаются резко при заполнении)  
- **В SwiftUI** — эквивалент — `StrokeStyle(lineCap: .round)` в `Path.stroke(style:)`  
- **Документируйте** — пишите комментарий «lineCap = .round — для плавных концов линий в прогресс-баре»

**Короткий итог 2026**:
> `CGLineCap` определяет **форму концов линий** при обводке пути в Core Graphics и `CAShapeLayer`.  
> В 2026 году:  
> - `.butt` — по умолчанию, самый точный и резкий  
> - `.round` — **самый популярный** и визуально приятный (выбирайте его в 90% случаев)  
> - `.square` — только для специфического стиля  
> Это **маленькое**, но **очень заметное** свойство, которое сильно влияет на эстетику толстых линий и прогресс-индикаторов.

Удачи с мягкими и профессионально выглядящими концами линий в твоём приложении! 📏✨