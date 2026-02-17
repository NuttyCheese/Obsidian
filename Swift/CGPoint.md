**CGPoint** — это простая структура из **Core Graphics** (импортируется через `import UIKit` / `import CoreGraphics`), которая описывает **двухмерную точку** в пространстве координат (x, y).

Это один из самых часто используемых типов в [[iOS]]/macOS-разработке 2026 года — практически везде, где нужна позиция: жесты, анимации, расположение вью, рендеринг, AR/Vision и т.д.

### Основное определение

```swift
public struct CGPoint {
    public var x: CGFloat
    public var y: CGFloat
    
    public init(x: CGFloat, y: CGFloat)
    public init()           // CGPoint.zero
}
```

- **CGFloat** — основной тип координат ([[Double]] на 64-битных системах)  
- **[[Value Type]]** ([[struct]]) — копируется по значению, полностью **[[Sendable]]**, безопасен в [[Swift Concurrency]]
- **[[Hashable]]**, **[[Equatable]]**, **[[Codable]]** (с iOS 11+)  
- **ExpressibleByArrayLiteral** (с iOS 16+): `[10.0, 20.0]` → CGPoint(x: 10, y: 20)

### Ключевые отличия CGPoint vs CGRect vs CGSize

| Тип            | Что описывает                    | x/y есть? | width/height есть? | Самый частый сценарий в 2026                   |
| -------------- | -------------------------------- | --------- | ------------------ | ---------------------------------------------- |
| **CGPoint**    | Только позиция (координаты)      | **Да**    | Нет                | Центр вью, translation жеста, позиция анимации |
| **[[CGSize]]** | Только размеры (ширина × высота) | Нет       | **Да**             | Размер вью, изображения, контента              |
| **[[CGRect]]** | Позиция + размер                 | **Да**    | **Да**             | frame, bounds, drawing rect                    |

### Самые частые сценарии использования CGPoint в 2026 году

1. **Позиционирование вью / слоёв / нодов**

```swift
profileImageView.center = CGPoint(x: view.bounds.midX, y: view.bounds.midY)
layer.position = CGPoint(x: 100, y: 200)
node.position = CGPoint(x: 0, y: 0)  // SpriteKit / SceneKit
```

2. **Жесты ([[UIPanGestureRecognizer]], [[UIPinchGestureRecognizer]])**

```swift
@objc func handlePan(_ gesture: UIPanGestureRecognizer) {
    let translation = gesture.translation(in: view)
    view.center = CGPoint(x: view.center.x + translation.x, y: view.center.y + translation.y)
    gesture.setTranslation(.zero, in: view)
}
```

3. **Анимации / CAAnimation**

```swift
let animation = CABasicAnimation(keyPath: "position")
animation.fromValue = NSValue(cgPoint: startPoint)
animation.toValue = NSValue(cgPoint: endPoint)
animation.duration = 0.5
layer.add(animation, forKey: "move")
```

4. **Расчёт расстояния / векторов / нормализация**

```swift
extension CGPoint {
    func distance(to other: CGPoint) -> CGFloat {
        hypot(x - other.x, y - other.y)
    }
    
    func normalized() -> CGPoint {
        let len = distance(to: .zero)
        return len > 0 ? CGPoint(x: x / len, y: y / len) : .zero
    }
    
    static func + (lhs: CGPoint, rhs: CGPoint) -> CGPoint {
        CGPoint(x: lhs.x + rhs.x, y: lhs.y + rhs.y)
    }
    
    static func * (lhs: CGPoint, rhs: CGFloat) -> CGPoint {
        CGPoint(x: lhs.x * rhs, y: lhs.y * rhs)
    }
}
```

5. **В [[SwiftUI]]** — через GeometryReader или .offset / .position

```swift
GeometryReader { geometry in
    Circle()
        .frame(width: 50, height: 50)
        .position(CGPoint(x: geometry.size.width / 2, y: geometry.size.height / 2))
}
```

6. **Vision / ARKit / hit-test**

```swift
let point = observation.boundingBox.origin  // CGPoint в normalized координатах
let converted = view.convert(point, from: view.bounds)  // в экранные координаты
```

### Лучшие практики работы с CGPoint в Swift 2026

- **bounds.midX / midY** — удобный способ получить центр вью  
- **Читай позицию в viewDidLayoutSubviews()** — до этого frame может быть неверным  
- **Используй safeAreaInsets** — для правильного позиционирования относительно краёв экрана  
- **@MainActor** — все операции с CGPoint в UI-контексте — на главном акторе  
- **Swift 6 strict concurrency** — CGPoint полностью Sendable (value type)  
- **Тестирование** — XCTAssertEqual(point1, point2, accuracy: 0.001) для floating-point  
- **Документируйте** — пиши комментарий «CGPoint — позиция в экранных координатах»  
- **Не путай с CGPoint в других системах** — в Metal/SceneKit координаты могут быть в NDC или world space

**Короткий девиз 2026**:
> «CGPoint — это когда тебе нужна **только позиция** (x, y) без размеров.  
> В 2026 году это **основной** тип для центрирования, жестов, анимаций, hit-test и позиционирования в UIKit/SwiftUI/SceneKit.  
> bounds.midX/midY — твой лучший друг для расчёта центра.»
