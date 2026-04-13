#uikit #uidynamics #physics #item-behavior #ios #swift #collision

---

## UIDynamicItemBehavior — Настройка физических свойств элементов

### Определение

**`UIDynamicItemBehavior`** — это класс в [[UIKit]], наследующий от [[UIDynamicBehavior]], который позволяет **настраивать индивидуальные физические свойства** элементов (например, трение, упругость, плотность, разрешение вращения). В отличие от других поведений (гравитация, столкновения), `UIDynamicItemBehavior` не добавляет новых сил, а **модифицирует** то, как элементы реагируют на существующие .

Без этого поведения все элементы имеют стандартные физические параметры. `UIDynamicItemBehavior` даёт тонкий контроль над каждым объектом в симуляции.

### Зачем это знать iOS-разработчику?

1.  **Реалистичное поведение:** Настройка упругости (отскока) и трения для разных типов объектов.
2.  **Контроль вращения:** Можно разрешить или запретить элементам вращаться при столкновениях.
3.  **Управление скоростью:** Прямое изменение линейной и угловой скорости объектов.
4.  **Индивидуальные настройки:** Разные элементы могут иметь разные физические параметры (например, лёгкий резиновый мяч vs тяжёлый стальной шар).

---

### Основные свойства

| Свойство                | Тип         | Описание                                                                                                                     |
| ----------------------- | ----------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **`elasticity`**        | `CGFloat`   | Упругость (отскок) при столкновениях. Диапазон: `0.0` (абсолютно неупругий) — `1.0` (идеально упругий). По умолчанию: `0.0`. |
| **`friction`**          | `CGFloat`   | Трение скольжения при контакте с другими элементами. По умолчанию: `0.0`.                                                    |
| **`density`**           | `CGFloat`   | Плотность, влияет на массу и инерцию. Чем выше плотность, тем сложнее изменить движение. По умолчанию: `1.0`.                |
| **`resistance`**        | `CGFloat`   | Сопротивление движению (линейное демпфирование). Замедляет объект со временем. По умолчанию: `0.0`.                          |
| **`angularResistance`** | `CGFloat`   | Сопротивление вращению (угловое демпфирование). По умолчанию: `0.0`.                                                         |
| **`allowsRotation`**    | `Bool`      | Разрешает или запрещает вращение элемента при столкновениях. По умолчанию: `true`.                                           |
| **`charge`**            | [[CGFloat]] | Электрический заряд (влияет на `UIFieldBehavior`). По умолчанию: `0.0`.                                                      |
| **`isAnchored`**        | [[Bool]]    | Если `true`, элемент закреплён (неподвижен). По умолчанию: `false`.                                                          |

---

### Методы управления скоростью

| Метод | Описание |
|---|---|
| **`addLinearVelocity(_:for:)`** | Добавляет линейную скорость элементу (точки/сек). |
| **`linearVelocity(for:)`** | Возвращает текущую линейную скорость элемента. |
| **`addAngularVelocity(_:for:)`** | Добавляет угловую скорость (радианы/сек). |
| **`angularVelocity(for:)`** | Возвращает текущую угловую скорость элемента. |

---

### Базовый пример: Разные физические свойства

```swift
import UIKit

class PhysicsPropertiesViewController: UIViewController {
    var animator: UIDynamicAnimator!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        animator = UIDynamicAnimator(referenceView: view)
        
        // Упругий мяч (хорошо отскакивает)
        let elasticBall = createBall(at: CGPoint(x: 80, y: 100), color: .systemRed)
        // Тяжёлый и неупругий шар (почти не отскакивает)
        let heavyBall = createBall(at: CGPoint(x: view.bounds.width - 80, y: 100), color: .systemBlue)
        
        // Гравитация для обоих
        let gravity = UIGravityBehavior(items: [elasticBall, heavyBall])
        animator.addBehavior(gravity)
        
        // Столкновения с границами
        let collision = UICollisionBehavior(items: [elasticBall, heavyBall])
        collision.translatesReferenceBoundsIntoBoundary = true
        animator.addBehavior(collision)
        
        // Настройка физических свойств для упругого мяча
        let elasticBehavior = UIDynamicItemBehavior(items: [elasticBall])
        elasticBehavior.elasticity = 0.9   // Сильный отскок
        elasticBehavior.friction = 0.2
        elasticBehavior.density = 0.5      // Лёгкий
        animator.addBehavior(elasticBehavior)
        
        // Настройка для тяжёлого шара
        let heavyBehavior = UIDynamicItemBehavior(items: [heavyBall])
        heavyBehavior.elasticity = 0.1     // Почти не отскакивает
        heavyBehavior.friction = 0.8
        heavyBehavior.density = 5.0        // Тяжёлый
        animator.addBehavior(heavyBehavior)
    }
    
    func createBall(at point: CGPoint, color: UIColor) -> UIView {
        let ball = UIView(frame: CGRect(x: point.x, y: point.y, width: 50, height: 50))
        ball.backgroundColor = color
        ball.layer.cornerRadius = 25
        view.addSubview(ball)
        return ball
    }
}
```

---

### Контроль вращения

```swift
class RotationControlViewController: UIViewController {
    var animator: UIDynamicAnimator!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        animator = UIDynamicAnimator(referenceView: view)
        
        // Квадрат, который может вращаться
        let rotatingSquare = createSquare(at: CGPoint(x: 100, y: 100), color: .systemGreen)
        // Квадрат, который НЕ вращается
        let fixedSquare = createSquare(at: CGPoint(x: 250, y: 100), color: .systemOrange)
        
        let gravity = UIGravityBehavior(items: [rotatingSquare, fixedSquare])
        animator.addBehavior(gravity)
        
        let collision = UICollisionBehavior(items: [rotatingSquare, fixedSquare])
        collision.translatesReferenceBoundsIntoBoundary = true
        animator.addBehavior(collision)
        
        // Разрешаем вращение для первого
        let rotateBehavior = UIDynamicItemBehavior(items: [rotatingSquare])
        rotateBehavior.allowsRotation = true
        rotateBehavior.elasticity = 0.8
        animator.addBehavior(rotateBehavior)
        
        // Запрещаем вращение для второго
        let noRotateBehavior = UIDynamicItemBehavior(items: [fixedSquare])
        noRotateBehavior.allowsRotation = false
        noRotateBehavior.elasticity = 0.8
        animator.addBehavior(noRotateBehavior)
    }
    
    func createSquare(at point: CGPoint, color: UIColor) -> UIView {
        let square = UIView(frame: CGRect(x: point.x, y: point.y, width: 60, height: 60))
        square.backgroundColor = color
        view.addSubview(square)
        return square
    }
}
```

---

### Управление скоростью: Толчок в определённом направлении

```swift
class VelocityControlViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var itemBehavior: UIDynamicItemBehavior!
    var ball: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        ball = createBall(at: CGPoint(x: view.bounds.midX, y: view.bounds.midY), color: .systemPurple)
        
        animator = UIDynamicAnimator(referenceView: view)
        
        let collision = UICollisionBehavior(items: [ball])
        collision.translatesReferenceBoundsIntoBoundary = true
        animator.addBehavior(collision)
        
        itemBehavior = UIDynamicItemBehavior(items: [ball])
        itemBehavior.elasticity = 0.9
        itemBehavior.friction = 0.2
        animator.addBehavior(itemBehavior)
        
        // Добавляем кнопки для управления
        addControlButtons()
    }
    
    func addControlButtons() {
        let upButton = UIButton(frame: CGRect(x: 50, y: view.bounds.height - 150, width: 80, height: 50))
        upButton.setTitle("↑ Вверх", for: .normal)
        upButton.backgroundColor = .systemBlue
        upButton.addTarget(self, action: #selector(pushUp), for: .touchUpInside)
        view.addSubview(upButton)
        
        let rightButton = UIButton(frame: CGRect(x: 150, y: view.bounds.height - 150, width: 80, height: 50))
        rightButton.setTitle("→ Вправо", for: .normal)
        rightButton.backgroundColor = .systemBlue
        rightButton.addTarget(self, action: #selector(pushRight), for: .touchUpInside)
        view.addSubview(rightButton)
    }
    
    @objc func pushUp() {
        // Получаем текущую скорость
        var velocity = itemBehavior.linearVelocity(for: ball)
        velocity.y -= 300  // Добавляем импульс вверх
        itemBehavior.addLinearVelocity(velocity, for: ball)
    }
    
    @objc func pushRight() {
        var velocity = itemBehavior.linearVelocity(for: ball)
        velocity.x += 300
        itemBehavior.addLinearVelocity(velocity, for: ball)
    }
    
    func createBall(at point: CGPoint, color: UIColor) -> UIView {
        let ball = UIView(frame: CGRect(x: point.x, y: point.y, width: 50, height: 50))
        ball.backgroundColor = color
        ball.layer.cornerRadius = 25
        view.addSubview(ball)
        return ball
    }
}
```

---

### Демпфирование: Замедление со временем

```swift
class DampingViewController: UIViewController {
    var animator: UIDynamicAnimator!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        animator = UIDynamicAnimator(referenceView: view)
        
        // Обычный мяч (без демпфирования)
        let normalBall = createBall(at: CGPoint(x: 80, y: 100), color: .systemRed)
        // Мяч с высоким сопротивлением
        let dampedBall = createBall(at: CGPoint(x: view.bounds.width - 80, y: 100), color: .systemBlue)
        
        // Даём начальный толчок
        let push = UIPushBehavior(items: [normalBall, dampedBall], mode: .instantaneous)
        push.angle = CGFloat.pi / 4  // 45 градусов
        push.magnitude = 5.0
        animator.addBehavior(push)
        
        let collision = UICollisionBehavior(items: [normalBall, dampedBall])
        collision.translatesReferenceBoundsIntoBoundary = true
        animator.addBehavior(collision)
        
        // Обычное поведение (без сопротивления)
        let normalBehavior = UIDynamicItemBehavior(items: [normalBall])
        normalBehavior.resistance = 0.0
        animator.addBehavior(normalBehavior)
        
        // Высокое сопротивление — мяч быстро остановится
        let dampedBehavior = UIDynamicItemBehavior(items: [dampedBall])
        dampedBehavior.resistance = 5.0
        animator.addBehavior(dampedBehavior)
    }
    
    func createBall(at point: CGPoint, color: UIColor) -> UIView {
        let ball = UIView(frame: CGRect(x: point.x, y: point.y, width: 50, height: 50))
        ball.backgroundColor = color
        ball.layer.cornerRadius = 25
        view.addSubview(ball)
        return ball
    }
}
```

---

### Закреплённые элементы (isAnchored)

```swift
class AnchoredViewController: UIViewController {
    var animator: UIDynamicAnimator!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        animator = UIDynamicAnimator(referenceView: view)
        
        // Подвижный шар
        let movingBall = createBall(at: CGPoint(x: 100, y: 100), color: .systemRed)
        // Закреплённый шар (неподвижный)
        let anchoredBall = createBall(at: CGPoint(x: view.bounds.midX, y: 200), color: .systemGray)
        
        let gravity = UIGravityBehavior(items: [movingBall, anchoredBall])
        animator.addBehavior(gravity)
        
        let collision = UICollisionBehavior(items: [movingBall, anchoredBall])
        collision.translatesReferenceBoundsIntoBoundary = true
        animator.addBehavior(collision)
        
        // Закрепляем второй шар
        let anchoredBehavior = UIDynamicItemBehavior(items: [anchoredBall])
        anchoredBehavior.isAnchored = true  // Теперь неподвижен
        animator.addBehavior(anchoredBehavior)
    }
    
    func createBall(at point: CGPoint, color: UIColor) -> UIView {
        let ball = UIView(frame: CGRect(x: point.x, y: point.y, width: 50, height: 50))
        ball.backgroundColor = color
        ball.layer.cornerRadius = 25
        view.addSubview(ball)
        return ball
    }
}
```

---

### Лучшие практики

1.  **Разделяйте поведения для разных типов объектов:** Создавайте отдельные `UIDynamicItemBehavior` для каждого логического типа элемента (мячи, камни, стены).
2.  **Используйте `resistance` для создания "вязкой" среды:** Высокое сопротивление имитирует движение в жидкости.
3.  **Контролируйте вращение:** Для падающих блоков в аркадах часто отключают вращение (`allowsRotation = false`), чтобы поведение было предсказуемее.
4.  **Проверяйте скорость перед добавлением импульса:** Используйте `linearVelocity(for:)` для расчёта необходимого изменения скорости.
5.  **Комбинируйте с `action` для обратной связи:** Можно отслеживать скорость элемента в каждом кадре.

```swift
itemBehavior.action = { [weak self] in
    guard let self = self else { return }
    let velocity = itemBehavior.linearVelocity(for: self.ball)
    print("Скорость: dx=\(velocity.x), dy=\(velocity.y)")
}
```

---

### Итог

**`UIDynamicItemBehavior`** — это незаменимый инструмент для тонкой настройки физического поведения объектов в [[UIKit]] Dynamics. Он позволяет:

1.  **Настраивать упругость, трение, плотность** каждого элемента индивидуально.
2.  **Контролировать вращение** и закреплять объекты.
3.  **Управлять скоростью** (линейной и угловой) в рантайме.
4.  **Создавать разнообразные физические материалы** (резина, сталь, лёд, вязкая жидкость).

Без этого поведения все элементы вели бы себя одинаково. `UIDynamicItemBehavior` даёт полный контроль над физикой каждого объекта .