**Point** в Swift чаще всего означает **структуру `CGPoint`** из **Core Graphics** — это базовый тип для представления **двухмерной точки** в пространстве (x, y).

По состоянию на 2026 год `CGPoint` остаётся **одним из самых часто используемых** типов в iOS/macOS-разработке, особенно при работе с UIKit, SwiftUI, Core Animation, Core Graphics, Metal, SpriteKit, SceneKit, ARKit и Vision.

### Ключевые характеристики CGPoint (2026 актуальные)

```swift
public struct CGPoint {
    public var x: CGFloat
    public var y: CGFloat
    
    public init(x: CGFloat, y: CGFloat)
    public init()  // (0, 0)
}
```

- **CGFloat** — основной тип координат (Float на 32-бит, Double на 64-бит)  
- **Value type** (struct) — копируется по значению, Sendable, безопасен в Swift Concurrency  
- **Hashable**, **Equatable**, **Codable** (с iOS 11+)  
- **ExpressibleByArrayLiteral** (с iOS 16+ / macOS 13+) — `[1.0, 2.0]` → CGPoint(x: 1, y: 2)

### Самые популярные сценарии использования CGPoint в 2026

| Сценарий                                      | Пример кода (2026 стиль) | Почему CGPoint всё ещё must-have |
|-----------------------------------------------|---------------------------|-----------------------------------|
| **Позиционирование UIView / CALayer**         | `view.frame.origin = CGPoint(x: 100, y: 200)` | UIKit/AppKit работают только с CGPoint/CGRect |
| **SwiftUI offset / position**                 | `.offset(CGPoint(x: 50, y: 30))` | SwiftUI использует CGPoint под капотом |
| **Жесты (UIPanGestureRecognizer)**            | `let translation = gesture.translation(in: view)` | Возвращает CGPoint |
| **Core Graphics / UIGraphicsContext**         | `context.move(to: CGPoint(x: 0, y: 0))` | Низкоуровневая графика |
| **Metal / SpriteKit / SceneKit**              | `node.position = CGPoint(x: 100, y: 200)` | Координаты в 2D-пространстве |
| **Vision / ARKit**                            | `let point = observation.boundingBox.origin` | Результаты детекции |
| **Анимации / CAAnimation**                    | `animation.fromValue = NSValue(cgPoint: startPoint)` | Core Animation ожидает NSValue с CGPoint |
| **Расчёт расстояния / векторов**              | `let distance = point1.distance(to: point2)` | Расширение через extension |

### Полезные расширения CGPoint (самые популярные в 2026)

```swift
extension CGPoint {
    // Расстояние до другой точки
    func distance(to other: CGPoint) -> CGFloat {
        hypot(x - other.x, y - other.y)
    }
    
    // Нормализованный вектор
    func normalized() -> CGPoint {
        let len = distance(to: .zero)
        return len > 0 ? CGPoint(x: x / len, y: y / len) : .zero
    }
    
    // Сложение точек (векторное)
    static func + (lhs: CGPoint, rhs: CGPoint) -> CGPoint {
        CGPoint(x: lhs.x + rhs.x, y: lhs.y + rhs.y)
    }
    
    static func - (lhs: CGPoint, rhs: CGPoint) -> CGPoint {
        CGPoint(x: lhs.x - rhs.x, y: lhs.y - rhs.y)
    }
    
    // Умножение на скаляр
    static func * (lhs: CGPoint, rhs: CGFloat) -> CGPoint {
        CGPoint(x: lhs.x * rhs, y: lhs.y * rhs)
    }
    
    // Удобные константы
    static let zero = CGPoint.zero
    static let center = CGPoint(x: 0.5, y: 0.5) // для якорных точек
}
```

### Лучшие практики CGPoint в Swift 2026

- **Всегда используй CGPoint вместо CGPointMake** — `CGPointMake` deprecated с iOS 9  
- **SwiftUI** — предпочитай `.offset(x:y:)` вместо `.offset(CGPoint)` (но оба работают)  
- **@MainActor** — все операции с UIView/CALayer — на главном акторе  
- **Swift 6 strict concurrency** — `CGPoint` полностью Sendable — безопасен для передачи между задачами  
- **Тестирование** — используй `XCTAssertEqual(point1, point2, accuracy: 0.001)` для floating-point  
- **Документируйте** — пиши комментарий «CGPoint — позиция в экранных координатах»  
- **Не путай с CGPoint в других системах** — в Metal/SceneKit координаты могут быть в другом пространстве (NDC, world space)

**Короткий девиз 2026**:
> «CGPoint — это когда тебе нужно описать точку на экране или в 2D-пространстве.  
> В 2026 году это **основной** тип для позиций, жестов, анимаций и графики в UIKit/AppKit/SwiftUI.  
> Работай с ним как с обычной структурой — он лёгкий, Sendable и безопасный.»

Удачи с точным и адаптивным позиционированием в Swift! 📍