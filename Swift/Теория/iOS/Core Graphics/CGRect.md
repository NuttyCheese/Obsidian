**CGRect** — это базовая структура из **[[Core Graphics]]** (импортируется через `import` [[UIKit]] / `import CoreGraphics`), которая описывает **прямоугольник** в двухмерном пространстве. Она состоит из двух частей:

- **origin** — точка левого верхнего угла (тип [[CGPoint]])
- **size** — ширина и высота (тип [[CGSize]])

```swift
public struct CGRect {
    public var origin: CGPoint
    public var size: CGSize
    
    public init()                    // → CGRect.zero
    public init(x: CGFloat, y: CGFloat, width: CGFloat, height: CGFloat)
    public init(origin: CGPoint, size: CGSize)
}
```

### Ключевые отличия CGRect vs bounds vs frame (самое важное в 2026 году)

| Свойство       | Что возвращает                                    | origin относительно чего?         | Меняется ли при transform/rotation?  | Когда использовать в 2026                                                 |
| -------------- | ------------------------------------------------- | --------------------------------- | ------------------------------------ | ------------------------------------------------------------------------- |
| **[[bounds]]** | CGRect с origin = (0, 0) и size = ширина × высота | Собственная система координат вью | **Нет** (transform не влияет)        | Когда нужны **реальные внутренние размеры** вью                           |
| **[[frame]]**  | CGRect в системе координат **супервида**          | Супервида                         | **Да** (сдвигается и поворачивается) | Когда нужна **позиция и размер** относительно родителя                    |
| **CGRect**     | Произвольный прямоугольник (любой origin + size)  | Зависит от контекста              | Зависит от контекста                 | Универсальный тип для любых прямоугольников (drawing, hit-test, clipping) |

### Самые частые и важные сценарии использования CGRect в 2026

1. **Работа с [[frame]] / [[bounds]] вью**

```swift
override func viewDidLayoutSubviews() {
    super.viewDidLayoutSubviews()
    
    // Актуальный frame относительно супервида
    let frame = profileImageView.frame
    
    // Актуальный bounds (внутренние размеры)
    let bounds = profileImageView.bounds
    
    // Пример: круглая аватарка
    profileImageView.layer.cornerRadius = min(bounds.width, bounds.height) / 2
    
    // Shadow path по актуальному размеру
    profileImageView.layer.shadowPath = UIBezierPath(roundedRect: bounds, cornerRadius: 60).cgPath
}
```

2. **Создание и модификация прямоугольников**

```swift
let rect = CGRect(x: 20, y: 40, width: 300, height: 200)

// Встроенные методы (очень удобные в 2026)
let insetRect = rect.inset(by: UIEdgeInsets(top: 10, left: 10, bottom: 10, right: 10))
let integralRect = rect.integral              // Выравнивание по пикселям (убирает дробные значения)
let standardized = rect.standardized          // Нормализует (убирает отрицательные размеры)
let unionRect = rect.union(anotherRect)       // Объединение двух прямоугольников
let intersection = rect.intersection(anotherRect)  // Пересечение
```

3. **Проверка пересечения / содержания**

```swift
if rect.contains(point) { 
    // точка внутри прямоугольника
}

if rect.intersects(anotherRect) {
    // прямоугольники пересекаются
}
```

4. **В [[SwiftUI]]** — через GeometryReader или .frame

```swift
GeometryReader { geometry in
    Rectangle()
        .frame(width: geometry.size.width * 0.8, height: geometry.size.height * 0.6)
        .position(x: geometry.size.width / 2, y: geometry.size.height / 2)
}
```

5. **Расчёт безопасной области / отступов**

```swift
let safeBounds = view.bounds.inset(by: view.safeAreaInsets)
```

### Полезные расширения для CGRect (очень популярны в 2026)

```swift
extension CGRect {
    var center: CGPoint {
        CGPoint(x: midX, y: midY)
    }
    
    var midX: CGFloat { origin.x + size.width / 2 }
    var midY: CGFloat { origin.y + size.height / 2 }
    
    func insetBy(dx: CGFloat, dy: CGFloat) -> CGRect {
        inset(by: UIEdgeInsets(top: dy, left: dx, bottom: dy, right: dx))
    }
    
    func scaled(by scale: CGFloat) -> CGRect {
        CGRect(x: origin.x * scale, y: origin.y * scale,
               width: size.width * scale, height: size.height * scale)
    }
    
    func offsetBy(dx: CGFloat, dy: CGFloat) -> CGRect {
        offsetBy(dx: dx, dy: dy)
    }
}
```

### Лучшие практики работы с CGRect в Swift 2026

- **Читай frame/bounds в [[viewDidLayoutSubviews]]() / [[layoutSubviews]]()** — до этого размеры обычно неверные  
- **bounds.origin всегда (0, 0)** — используй для внутренних размеров  
- **frame.origin** — позиция относительно супервида (может быть отрицательным)  
- **Используй [[safeAreaLayoutGuide]] / readableContentGuide** — вместо ручного расчёта отступов  
- **integral / standardized** — используй для выравнивания по пикселям (убирает субпиксельные ошибки)  
- **[[@MainActor]]** — все операции с CGRect в UI-контексте — на главном акторе  
- **Swift 6 strict concurrency** — CGRect полностью [[Sendable]] ([[value type]])  
- **Тестирование** — XCTAssertEqual(rect1, rect2, accuracy: 0.001) для floating-point  
- **Документируйте** — пиши комментарий «CGRect — актуальный frame после Auto Layout»

**Короткий девиз 2026**:
> «CGRect — это когда тебе нужна **полная информация о прямоугольнике**: и позиция (origin), и размер (size).  
> bounds — только размер (origin всегда 0,0), frame — позиция + размер относительно супервида.  
> Читай в [[viewDidLayoutSubviews]], используй integral/standardized для точности по пикселям.»
