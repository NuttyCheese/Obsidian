#uikit #uidynamics #push #physics #ios #swift

---

## UIPushBehavior — Сила толчка в физической анимации

### Определение

**`UIPushBehavior`** — это класс в [[UIKit]], наследующий от [[UIDynamicBehavior]], который прикладывает **силу толчка** к элементам. Толчок может быть **непрерывным** (постоянно действующая сила, например, ветер или течение) или **мгновенным** (однократный импульс, например, удар по мячу) .

Это поведение идеально подходит для создания эффектов выстрела, броска, движения под действием ветра, реактивного движения и любых других сценариев, где объект получает ускорение в определённом направлении.

### Зачем это знать iOS-разработчику?

1.  **Выстрелы и броски:** Снаряды, мячи, снежки, летающие объекты.
2.  **Постоянные силы:** Ветер, течение, гравитация (хотя для гравитации есть [[UIGravityBehavior]]).
3.  **Реактивное движение:** Имитация работы ракетного двигателя или пропеллера.
4.  **Интерактивные элементы:** Смахивание карточек (Tinder-like), ускорение объектов жестом.
5.  **Игровая механика:** Пинбол, бильярд, футбол, аркады.

---

### Архитектура и ключевые свойства

#### Инициализация

```swift
init(items: [UIDynamicItem], mode: UIPushBehaviorMode)
```

#### Режимы толчка

| Режим | Описание |
|---|---|
| **`.continuous`** | Сила прикладывается **непрерывно** в каждом кадре. Объект постоянно ускоряется. |
| **`.instantaneous`** | Сила прикладывается **один раз** (мгновенный импульс). Объект получает начальную скорость и дальше движется по инерции. |

#### Основные свойства

| Свойство            | Тип          | Описание                                                                                    |
| ------------------- | ------------ | ------------------------------------------------------------------------------------------- |
| **`active`**        | [[Bool]]     | Включен ли толчок. Для `.instantaneous` автоматически становится `false` после применения.  |
| **`angle`**         | `CGFloat`    | Угол направления толчка в радианах.                                                         |
| **`magnitude`**     | [[CGFloat]]  | Сила толчка (масштаб вектора).                                                              |
| **`pushDirection`** | [[CGVector]] | Вектор направления толчка (альтернатива `angle`).                                           |
| **`targetOffset`**  | [[UIOffset]] | Смещение точки приложения силы относительно центра элемента (по умолчанию `.zero` — центр). |

---

### Базовый пример: Мгновенный толчок (удар)

```swift
import UIKit

class PushViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var push: UIPushBehavior!
    var ball: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Создаем мяч
        ball = UIView(frame: CGRect(x: 100, y: 200, width: 60, height: 60))
        ball.backgroundColor = .systemBlue
        ball.layer.cornerRadius = 30
        view.addSubview(ball)
        
        animator = UIDynamicAnimator(referenceView: view)
        
        // Мгновенный толчок
        push = UIPushBehavior(items: [ball], mode: .instantaneous)
        push.angle = CGFloat.pi / 4  // 45 градусов
        push.magnitude = 2.0
        animator.addBehavior(push)
        
        // Столкновения с границами
        let collision = UICollisionBehavior(items: [ball])
        collision.translatesReferenceBoundsIntoBoundary = true
        animator.addBehavior(collision)
        
        // Физические свойства (упругость, трение)
        let properties = UIDynamicItemBehavior(items: [ball])
        properties.elasticity = 0.8
        properties.friction = 0.2
        animator.addBehavior(properties)
        
        // Кнопка для нового толчка
        let button = UIButton(frame: CGRect(x: 50, y: view.bounds.height - 100, width: 100, height: 50))
        button.setTitle("Удар", for: .normal)
        button.backgroundColor = .systemRed
        button.addTarget(self, action: #selector(applyPush), for: .touchUpInside)
        view.addSubview(button)
    }
    
    @objc func applyPush() {
        push.active = true  // Применяем толчок
    }
}
```

---

### Непрерывный толчок (ветер)

```swift
class WindViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var wind: UIPushBehavior!
    var balls: [UIView] = []
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Добавляем несколько шариков
        for i in 0..<10 {
            let ball = UIView(frame: CGRect(x: 50 + i * 30, y: 100, width: 25, height: 25))
            ball.backgroundColor = .systemGreen
            ball.layer.cornerRadius = 12.5
            view.addSubview(ball)
            balls.append(ball)
        }
        
        animator = UIDynamicAnimator(referenceView: view)
        
        // Гравитация (тянет вниз)
        let gravity = UIGravityBehavior(items: balls)
        animator.addBehavior(gravity)
        
        // Столкновения с границами
        let collision = UICollisionBehavior(items: balls)
        collision.translatesReferenceBoundsIntoBoundary = true
        animator.addBehavior(collision)
        
        // Непрерывный ветер вправо
        wind = UIPushBehavior(items: balls, mode: .continuous)
        wind.pushDirection = CGVector(dx: 0.5, dy: 0.0)  // Вправо
        wind.magnitude = 0.3
        wind.active = true
        animator.addBehavior(wind)
        
        // Слайдер для регулировки силы ветра
        let slider = UISlider(frame: CGRect(x: 50, y: view.bounds.height - 100, width: view.bounds.width - 100, height: 30))
        slider.minimumValue = 0.0
        slider.maximumValue = 1.0
        slider.value = 0.3
        slider.addTarget(self, action: #selector(windChanged(_:)), for: .valueChanged)
        view.addSubview(slider)
    }
    
    @objc func windChanged(_ slider: UISlider) {
        wind.magnitude = CGFloat(slider.value)
    }
}
```

---

### Толчок с заданным вектором

```swift
class VectorPushViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var push: UIPushBehavior!
    var ball: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        ball = UIView(frame: CGRect(x: view.bounds.midX, y: view.bounds.midY, width: 50, height: 50))
        ball.backgroundColor = .systemPurple
        ball.layer.cornerRadius = 25
        view.addSubview(ball)
        
        animator = UIDynamicAnimator(referenceView: view)
        
        let collision = UICollisionBehavior(items: [ball])
        collision.translatesReferenceBoundsIntoBoundary = true
        animator.addBehavior(collision)
        
        let properties = UIDynamicItemBehavior(items: [ball])
        properties.elasticity = 0.9
        animator.addBehavior(properties)
        
        push = UIPushBehavior(items: [ball], mode: .instantaneous)
        
        // Добавляем жесты для управления направлением
        let tap = UITapGestureRecognizer(target: self, action: #selector(handleTap(_:)))
        view.addGestureRecognizer(tap)
    }
    
    @objc func handleTap(_ gesture: UITapGestureRecognizer) {
        let tapPoint = gesture.location(in: view)
        let ballCenter = ball.center
        
        // Вычисляем вектор от мяча к точке касания
        let dx = tapPoint.x - ballCenter.x
        let dy = tapPoint.y - ballCenter.y
        let length = sqrt(dx * dx + dy * dy)
        
        if length > 0 {
            let normalizedDx = dx / length
            let normalizedDy = dy / length
            
            push.pushDirection = CGVector(dx: normalizedDx, dy: normalizedDy)
            push.magnitude = 1.5
            push.active = true
        }
    }
}
```

---

### Прицеливание с визуальной обратной связью

```swift
class AimingViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var push: UIPushBehavior!
    var ball: UIView!
    var aimingLine: UIView?
    var startPoint: CGPoint?
    
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
        
        let properties = UIDynamicItemBehavior(items: [ball])
        properties.elasticity = 0.7
        animator.addBehavior(properties)
        
        push = UIPushBehavior(items: [ball], mode: .instantaneous)
        animator.addBehavior(push)
        
        // Жест перетаскивания для прицеливания
        let pan = UIPanGestureRecognizer(target: self, action: #selector(handlePan))
        view.addGestureRecognizer(pan)
    }
    
    @objc func handlePan(_ gesture: UIPanGestureRecognizer) {
        let location = gesture.location(in: view)
        
        switch gesture.state {
        case .began:
            startPoint = ball.center
            createAimingLine()
            
        case .changed:
            updateAimingLine(to: location)
            
        case .ended:
            applyPush(to: location)
            removeAimingLine()
            
        default: break
        }
    }
    
    func createAimingLine() {
        aimingLine = UIView(frame: .zero)
        aimingLine?.backgroundColor = .red
        view.addSubview(aimingLine!)
    }
    
    func updateAimingLine(to point: CGPoint) {
        guard let start = startPoint else { return }
        
        let dx = point.x - start.x
        let dy = point.y - start.y
        let length = sqrt(dx * dx + dy * dy)
        let angle = atan2(dy, dx)
        
        aimingLine?.frame = CGRect(x: start.x, y: start.y, width: length, height: 3)
        aimingLine?.center = CGPoint(x: start.x + dx / 2, y: start.y + dy / 2)
        aimingLine?.transform = CGAffineTransform(rotationAngle: angle)
    }
    
    func applyPush(to point: CGPoint) {
        guard let start = startPoint else { return }
        
        let dx = point.x - start.x
        let dy = point.y - start.y
        let distance = sqrt(dx * dx + dy * dy)
        
        // Чем дальше оттянули, тем сильнее удар
        let magnitude = min(distance / 100, 3.0)
        
        push.pushDirection = CGVector(dx: dx / distance, dy: dy / distance)
        push.magnitude = magnitude
        push.active = true
    }
    
    func removeAimingLine() {
        aimingLine?.removeFromSuperview()
        aimingLine = nil
    }
}
```

---

### Вращающий момент (torque)

```swift
class TorqueViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var push: UIPushBehavior!
    var square: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        square = UIView(frame: CGRect(x: 150, y: 200, width: 80, height: 80))
        square.backgroundColor = .systemTeal
        view.addSubview(square)
        
        animator = UIDynamicAnimator(referenceView: view)
        
        let collision = UICollisionBehavior(items: [square])
        collision.translatesReferenceBoundsIntoBoundary = true
        animator.addBehavior(collision)
        
        // Вращательный момент (толчок с отступом от центра)
        push = UIPushBehavior(items: [square], mode: .instantaneous)
        push.targetOffset = UIOffset(horizontal: 30, vertical: 0)  // Смещение вправо от центра
        push.angle = 0  // Горизонтальный толчок
        push.magnitude = 1.0
        animator.addBehavior(push)
        
        let button = UIButton(frame: CGRect(x: 50, y: view.bounds.height - 100, width: 150, height: 50))
        button.setTitle("Закрутить", for: .normal)
        button.backgroundColor = .systemBlue
        button.addTarget(self, action: #selector(applyTorque), for: .touchUpInside)
        view.addSubview(button)
    }
    
    @objc func applyTorque() {
        push.active = true
    }
}
```

---

### Лучшие практики

1.  **Для `.instantaneous` не забывайте устанавливать `active = true`:** После применения толчок автоматически деактивируется.
2.  **Используйте `.continuous` для постоянных сил:** Ветер, течение, "двигатель".
3.  **Комбинируйте с `UIDynamicItemBehavior`:** Для контроля массы (`density`), которая влияет на то, как объект реагирует на толчок.
4.  **Регулируйте `magnitude` в зависимости от контекста:** В играх часто масштабируют силу в зависимости от скорости жеста или расстояния.
5.  **Используйте `targetOffset` для создания вращения:** Смещение точки приложения силы создаёт момент вращения (torque).

```swift
// Пример: Чем тяжелее объект, тем сильнее толчок для одинакового эффекта
let properties = UIDynamicItemBehavior(items: [heavyBall])
let mass = properties.density
push.magnitude = baseForce * mass
```

---

### Итог

**`UIPushBehavior`** — это универсальный инструмент для приложения сил в UIKit Dynamics. Он позволяет:

1.  **Создавать мгновенные импульсы** (удары, выстрелы, броски).
2.  **Прикладывать непрерывные силы** (ветер, течение, двигатель).
3.  **Задавать направление** через угол или вектор.
4.  **Регулировать силу** (`magnitude`).
5.  **Создавать вращающий момент** через `targetOffset`.

Толчок — ключевой элемент для создания интерактивной физической среды, где объекты реагируют на действия пользователя или внешние силы .