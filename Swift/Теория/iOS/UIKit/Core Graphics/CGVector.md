#core-graphics #cgvector #math #physics #uikit #ios #swift

---

## CGVector — Вектор в [[Core Graphics]]

### Определение

**`CGVector`** — это структура в Core Graphics, представляющая **двумерный вектор** с компонентами `dx` (изменение по оси X) и `dy` (изменение по оси Y). Вектор описывает направление и величину (длину), но не положение в пространстве .

В отличие от [[CGPoint]] (который определяет **точку** с абсолютными координатами), `CGVector` определяет **смещение** или **силу**, приложенную к объекту.

### Зачем это знать iOS-разработчику?

1.  **UIKit Dynamics:** Векторы используются для задания направления гравитации, силы толчка ([[UIPushBehavior]]), скорости ([[UIDynamicItemBehavior]]).
2.  **Core Graphics:** Задание направления градиента ([[CGGradient]]), смещения в трансформациях ([[CGAffineTransform]]).
3.  **Физика и игры:** Скорость, ускорение, силы, импульсы.
4.  **Анимации:** Векторы перемещения в `UIView.animate`.
5.  **Математика:** Сложение, вычитание, масштабирование, нормализация, скалярное произведение.

---

### Создание CGVector

```swift
import CoreGraphics

// Стандартный инициализатор
let vector = CGVector(dx: 10.0, dy: 5.0)

// Нулевой вектор (нулевая длина)
let zero = CGVector(dx: 0, dy: 0)

// Из CGPoint (как вектор от начала координат)
let point = CGPoint(x: 3, y: 4)
let vectorFromPoint = CGVector(dx: point.x, dy: point.y)

// Из разности двух точек (вектор от A к B)
let start = CGPoint(x: 1, y: 2)
let end = CGPoint(x: 4, y: 6)
let diff = CGVector(dx: end.x - start.x, dy: end.y - start.y)
```

---

### Базовые операции

#### 1. **Сложение и вычитание векторов**

```swift
extension CGVector {
    static func + (lhs: CGVector, rhs: CGVector) -> CGVector {
        return CGVector(dx: lhs.dx + rhs.dx, dy: lhs.dy + rhs.dy)
    }
    
    static func - (lhs: CGVector, rhs: CGVector) -> CGVector {
        return CGVector(dx: lhs.dx - rhs.dx, dy: lhs.dy - rhs.dy)
    }
}

let v1 = CGVector(dx: 2, dy: 3)
let v2 = CGVector(dx: 1, dy: 4)
let sum = v1 + v2  // (3, 7)
let diff = v1 - v2 // (1, -1)
```

#### 2. **Умножение на скаляр**

```swift
extension CGVector {
    static func * (vector: CGVector, scalar: CGFloat) -> CGVector {
        return CGVector(dx: vector.dx * scalar, dy: vector.dy * scalar)
    }
    
    static func * (scalar: CGFloat, vector: CGVector) -> CGVector {
        return vector * scalar
    }
}

let v = CGVector(dx: 2, dy: 3)
let doubled = v * 2  // (4, 6)
let half = v * 0.5   // (1, 1.5)
```

#### 3. **Длина вектора (magnitude)**

```swift
extension CGVector {
    var magnitude: CGFloat {
        return sqrt(dx * dx + dy * dy)
    }
    
    var squaredMagnitude: CGFloat {
        return dx * dx + dy * dy
    }
}

let v = CGVector(dx: 3, dy: 4)
print(v.magnitude)  // 5.0
```

#### 4. **Нормализация (получение единичного вектора)**

```swift
extension CGVector {
    var normalized: CGVector {
        let len = magnitude
        guard len > 0 else { return .zero }
        return CGVector(dx: dx / len, dy: dy / len)
    }
}

let v = CGVector(dx: 5, dy: 5)
let unit = v.normalized  // (~0.707, ~0.707)
```

#### 5. **Скалярное произведение (dot product)**

```swift
extension CGVector {
    static func dot(_ lhs: CGVector, _ rhs: CGVector) -> CGFloat {
        return lhs.dx * rhs.dx + lhs.dy * rhs.dy
    }
}

let v1 = CGVector(dx: 1, dy: 0)
let v2 = CGVector(dx: 0, dy: 1)
let dot = CGVector.dot(v1, v2)  // 0 (перпендикулярны)
```

#### 6. **Угол между векторами**

```swift
extension CGVector {
    func angle(to other: CGVector) -> CGFloat {
        let dot = CGVector.dot(self, other)
        let magProduct = magnitude * other.magnitude
        guard magProduct > 0 else { return 0 }
        return acos(dot / magProduct)
    }
}

let v1 = CGVector(dx: 1, dy: 0)
let v2 = CGVector(dx: 0, dy: 1)
let angle = v1.angle(to: v2)  // π/2 (90 градусов)
```

---

### Использование в [[UIKit]] Dynamics

#### 1. **Гравитация ([[UIGravityBehavior]])**

```swift
import UIKit

class GravityVectorViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var gravity: UIGravityBehavior!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let ball = UIView(frame: CGRect(x: 100, y: 100, width: 50, height: 50))
        ball.backgroundColor = .systemBlue
        ball.layer.cornerRadius = 25
        view.addSubview(ball)
        
        animator = UIDynamicAnimator(referenceView: view)
        gravity = UIGravityBehavior(items: [ball])
        
        // Вектор гравитации: вниз-вправо
        gravity.gravityDirection = CGVector(dx: 0.5, dy: 1.0)
        gravity.magnitude = 1.0
        
        animator.addBehavior(gravity)
    }
}
```

#### 2. **Толчок (UIPushBehavior)**

```swift
class PushVectorViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var push: UIPushBehavior!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let box = UIView(frame: CGRect(x: 150, y: 200, width: 60, height: 60))
        box.backgroundColor = .systemRed
        view.addSubview(box)
        
        animator = UIDynamicAnimator(referenceView: view)
        
        // Мгновенный толчок с вектором (вверх-вправо)
        push = UIPushBehavior(items: [box], mode: .instantaneous)
        push.pushDirection = CGVector(dx: 0.8, dy: -0.6)
        push.magnitude = 2.0
        animator.addBehavior(push)
        
        let button = UIButton(frame: CGRect(x: 50, y: 500, width: 100, height: 50))
        button.setTitle("Толкнуть", for: .normal)
        button.backgroundColor = .systemGreen
        button.addTarget(self, action: #selector(applyPush), for: .touchUpInside)
        view.addSubview(button)
    }
    
    @objc func applyPush() {
        push.active = true
    }
}
```

#### 3. **Скорость элемента ([[UIDynamicItemBehavior]])**

```swift
class VelocityVectorViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var itemBehavior: UIDynamicItemBehavior!
    var ball: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        ball = UIView(frame: CGRect(x: 100, y: 300, width: 50, height: 50))
        ball.backgroundColor = .systemOrange
        ball.layer.cornerRadius = 25
        view.addSubview(ball)
        
        animator = UIDynamicAnimator(referenceView: view)
        
        let collision = UICollisionBehavior(items: [ball])
        collision.translatesReferenceBoundsIntoBoundary = true
        animator.addBehavior(collision)
        
        itemBehavior = UIDynamicItemBehavior(items: [ball])
        animator.addBehavior(itemBehavior)
        
        // Устанавливаем начальную скорость
        let velocity = CGVector(dx: 200, dy: -150)
        itemBehavior.addLinearVelocity(velocity, for: ball)
    }
}
```

---

### Использование в Core Graphics

#### 1. **Направление градиента**

```swift
import UIKit

class GradientViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let gradientLayer = CAGradientLayer()
        gradientLayer.frame = view.bounds
        
        // Вектор направления градиента: слева направо
        gradientLayer.startPoint = CGPoint(x: 0, y: 0.5)
        gradientLayer.endPoint = CGPoint(x: 1, y: 0.5)
        
        gradientLayer.colors = [UIColor.red.cgColor, UIColor.blue.cgColor]
        view.layer.addSublayer(gradientLayer)
    }
}
```

#### 2. **CGAffineTransform с вектором перемещения**

```swift
let translation = CGVector(dx: 100, dy: 50)
let transform = CGAffineTransform(translationX: translation.dx, y: translation.dy)
```

---

### Связь CGVector → [[CGPoint]]

```swift
extension CGVector {
    func toPoint() -> CGPoint {
        return CGPoint(x: dx, y: dy)
    }
}

extension CGPoint {
    func toVector() -> CGVector {
        return CGVector(dx: x, dy: y)
    }
}

let vector = CGVector(dx: 10, dy: 20)
let point = vector.toPoint()  // (10, 20)
```

---

### Игровой пример: отскок с нормалью

```swift
struct Collision {
    static func reflect(velocity: CGVector, normal: CGVector) -> CGVector {
        let dot = CGVector.dot(velocity, normal)
        let reflection = velocity - normal * (2 * dot)
        return reflection
    }
}

let incoming = CGVector(dx: 5, dy: -3)
let wallNormal = CGVector(dx: 0, dy: 1)  // стена снизу
let reflected = Collision.reflect(velocity: incoming, normal: wallNormal)
print(reflected)  // (5, 3)
```

---

### CGVectorZero и пустой вектор

```swift
let zeroVector = CGVector.zero  // (0, 0)
let isZero = zeroVector == CGVector(dx: 0, dy: 0)  // true
```

---

### Лучшие практики

1.  **Используйте `CGVector` для направления и силы, `CGPoint` для позиций.**
2.  **Для сложных векторных операций создавайте расширения.**
3.  **Нормализуйте векторы перед использованием в качестве направления.**
4.  **Проверяйте нулевую длину перед нормализацией** (деление на ноль).
5.  **В UIKit Dynamics векторные компоненты могут быть больше 1.0** (масштабируются `magnitude`).

```swift
// Проверка перед нормализацией
let direction = CGVector(dx: 0, dy: 0)
if direction.magnitude > 0 {
    let normalized = direction.normalized
} else {
    // обработка нулевого вектора
}
```

---

### Итог

**`CGVector`** — это фундаментальная структура для работы с двумерными векторами в iOS. Она позволяет:

1.  **Описывать направление и силу** в физических движках (UIKit Dynamics).
2.  **Вычислять скорость, ускорение, импульсы.**
3.  **Задавать гравитацию, толчки, силы.**
4.  **Работать с направлениями в Core Graphics.**
5.  **Выполнять математические операции** (сложение, вычитание, масштабирование, нормализация, скалярное произведение).

Понимание `CGVector` необходимо для создания физически правдоподобных анимаций, игр и интерактивных интерфейсов .