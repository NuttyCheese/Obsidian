#uikit #uidynamics #gravity #physics #ios #swift

---

## UIGravityBehavior — Сила гравитации в физической анимации

### Определение

**`UIGravityBehavior`** — это класс в [[UIKit]], наследующий от [[UIDynamicBehavior]], который применяет **силу гравитации** к элементам. Под действием этой силы объекты ускоряются в заданном направлении (по умолчанию — вниз), имитируя притяжение Земли или любой другой вектор .

Это одно из самых базовых и часто используемых поведений в [[UIDynamicAnimator]]. Оно позволяет создавать эффекты падения, наклонных плоскостей, "магнитного" притяжения в любую сторону.

### Зачем это знать iOS-разработчику?

1.  **Падающие объекты:** Создание реалистичного падения элементов интерфейса, карточек, мячей.
2.  **Наклонные плоскости:** Настройка вектора гравитации для имитации склона или наклонного экрана.
3.  **Магнитное притяжение:** Отрицательная гравитация ("антигравитация") может притягивать объекты вверх или к центру.
4.  **Игровая механика:** Физика прыжков, падения платформ, движения снарядов.
5.  **Комбинация с другими поведениями:** Работает в связке с [[UICollisionBehavior]], [[UIDynamicItemBehavior]], [[UIPushBehavior]].

---

### Архитектура и ключевые свойства

#### Инициализация

```swift
init(items: [UIDynamicItem])
```

#### Основные свойства

| Свойство               | Тип             | Описание                                                                          |
| ---------------------- | --------------- | --------------------------------------------------------------------------------- |
| **`gravityDirection`** | [[CGVector]]    | Вектор направления гравитации. По умолчанию: `CGVector(dx: 0.0, dy: 1.0)` — вниз. |
| **`angle`**            | [[CGFloat]]     | Угол направления гравитации в радианах (альтернатива `gravityDirection`).         |
| **`magnitude`**        | `CGFloat`       | Сила гравитации (масштаб вектора). По умолчанию: `1.0`.                           |
| **`action`**           | `(() -> Void)?` | Блок кода, вызываемый в каждом кадре анимации.                                    |

#### Методы управления элементами

```swift
func addItem(_ item: UIDynamicItem)
func removeItem(_ item: UIDynamicItem)
func items() -> [UIDynamicItem]
```

---

### Базовый пример: Падающий мяч

```swift
import UIKit

class GravityViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var gravity: UIGravityBehavior!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Создаем мяч
        let ball = UIView(frame: CGRect(x: 100, y: 100, width: 60, height: 60))
        ball.backgroundColor = .systemBlue
        ball.layer.cornerRadius = 30
        view.addSubview(ball)
        
        // Аниматор
        animator = UIDynamicAnimator(referenceView: view)
        
        // Гравитация (по умолчанию вниз)
        gravity = UIGravityBehavior(items: [ball])
        animator.addBehavior(gravity)
        
        // Добавляем столкновения, чтобы мяч не упал за пределы экрана
        let collision = UICollisionBehavior(items: [ball])
        collision.translatesReferenceBoundsIntoBoundary = true
        animator.addBehavior(collision)
    }
}
```

---

### Настройка направления и силы

#### 1. **Гравитация вниз с увеличенной силой**

```swift
gravity.gravityDirection = CGVector(dx: 0.0, dy: 1.0)
gravity.magnitude = 2.0  // В два раза сильнее обычной гравитации
```

#### 2. **Гравитация вверх (антигравитация)**

```swift
gravity.gravityDirection = CGVector(dx: 0.0, dy: -1.0)
gravity.magnitude = 1.5
```

#### 3. **Гравитация под углом 45 градусов**

```swift
// Через вектор
gravity.gravityDirection = CGVector(dx: 1.0, dy: 1.0)

// Через угол (в радианах)
gravity.angle = CGFloat.pi / 4  // 45 градусов
gravity.magnitude = 1.0
```

#### 4. **Горизонтальная гравитация (влево)**

```swift
gravity.gravityDirection = CGVector(dx: -1.0, dy: 0.0)
```

---

### Пример: Наклонная плоскость

```swift
class SlopedPlaneViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var gravity: UIGravityBehavior!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Создаем квадрат
        let box = UIView(frame: CGRect(x: 50, y: 100, width: 50, height: 50))
        box.backgroundColor = .systemOrange
        view.addSubview(box)
        
        animator = UIDynamicAnimator(referenceView: view)
        
        // Гравитация под углом (имитация наклонной плоскости)
        gravity = UIGravityBehavior(items: [box])
        gravity.gravityDirection = CGVector(dx: 0.8, dy: 1.0)  // Вниз и вправо
        gravity.magnitude = 1.5
        animator.addBehavior(gravity)
        
        // Столкновения с границами
        let collision = UICollisionBehavior(items: [box])
        collision.translatesReferenceBoundsIntoBoundary = true
        animator.addBehavior(collision)
        
        // Добавляем физические свойства (трение, упругость)
        let properties = UIDynamicItemBehavior(items: [box])
        properties.elasticity = 0.6
        properties.friction = 0.3
        animator.addBehavior(properties)
    }
}
```

---

### Магнитное притяжение к центру

```swift
class MagneticCenterViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var gravity: UIGravityBehavior!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Создаем несколько частиц
        let particles = (0..<20).map { _ -> UIView in
            let x = CGFloat.random(in: 0...view.bounds.width)
            let y = CGFloat.random(in: 0...view.bounds.height)
            let particle = UIView(frame: CGRect(x: x, y: y, width: 15, height: 15))
            particle.backgroundColor = .systemPurple
            particle.layer.cornerRadius = 7.5
            view.addSubview(particle)
            return particle
        }
        
        animator = UIDynamicAnimator(referenceView: view)
        
        // Гравитация к центру
        gravity = UIGravityBehavior(items: particles)
        
        // Вектор от позиции частицы к центру обновляется в action
        gravity.action = { [weak self] in
            guard let self = self else { return }
            for particle in particles {
                let center = self.view.center
                let dx = center.x - particle.center.x
                let dy = center.y - particle.center.y
                let distance = sqrt(dx * dx + dy * dy)
                if distance > 0 {
                    let normalizedDx = dx / distance
                    let normalizedDy = dy / distance
                    self.gravity.gravityDirection = CGVector(dx: normalizedDx, dy: normalizedDy)
                }
            }
        }
        
        animator.addBehavior(gravity)
        
        // Добавляем столкновения между частицами
        let collision = UICollisionBehavior(items: particles)
        collision.collisionMode = .items
        animator.addBehavior(collision)
    }
}
```

---

### Комбинация с [[UIAttachmentBehavior]] (эффект "резинки")

```swift
class GravityWithAttachmentViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var gravity: UIGravityBehavior!
    var attachment: UIAttachmentBehavior?
    var ball: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        ball = UIView(frame: CGRect(x: 100, y: 100, width: 50, height: 50))
        ball.backgroundColor = .systemRed
        ball.layer.cornerRadius = 25
        view.addSubview(ball)
        
        animator = UIDynamicAnimator(referenceView: view)
        
        // Гравитация тянет вниз
        gravity = UIGravityBehavior(items: [ball])
        gravity.gravityDirection = CGVector(dx: 0, dy: 1.0)
        animator.addBehavior(gravity)
        
        // Столкновения с границами
        let collision = UICollisionBehavior(items: [ball])
        collision.translatesReferenceBoundsIntoBoundary = true
        animator.addBehavior(collision)
        
        // Жест перетаскивания
        let pan = UIPanGestureRecognizer(target: self, action: #selector(handlePan))
        ball.addGestureRecognizer(pan)
        ball.isUserInteractionEnabled = true
    }
    
    @objc func handlePan(_ gesture: UIPanGestureRecognizer) {
        let location = gesture.location(in: view)
        
        switch gesture.state {
        case .began:
            attachment = UIAttachmentBehavior(item: ball, attachedToAnchor: location)
            attachment?.frequency = 2.0
            attachment?.damping = 0.5
            animator.addBehavior(attachment!)
            
        case .changed:
            attachment?.anchorPoint = location
            
        case .ended, .cancelled:
            if let attachment = attachment {
                animator.removeBehavior(attachment)
                self.attachment = nil
            }
            
        default: break
        }
    }
}
```

---

### Динамическое изменение силы гравитации

```swift
class VariableGravityViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var gravity: UIGravityBehavior!
    var slider: UISlider!
    var ball: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        ball = UIView(frame: CGRect(x: 100, y: 100, width: 60, height: 60))
        ball.backgroundColor = .systemGreen
        ball.layer.cornerRadius = 30
        view.addSubview(ball)
        
        animator = UIDynamicAnimator(referenceView: view)
        
        gravity = UIGravityBehavior(items: [ball])
        gravity.magnitude = 1.0
        animator.addBehavior(gravity)
        
        let collision = UICollisionBehavior(items: [ball])
        collision.translatesReferenceBoundsIntoBoundary = true
        animator.addBehavior(collision)
        
        // Слайдер для регулировки силы гравитации
        slider = UISlider(frame: CGRect(x: 50, y: view.bounds.height - 100, width: view.bounds.width - 100, height: 30))
        slider.minimumValue = 0.0
        slider.maximumValue = 5.0
        slider.value = 1.0
        slider.addTarget(self, action: #selector(gravityChanged), for: .valueChanged)
        view.addSubview(slider)
    }
    
    @objc func gravityChanged() {
        gravity.magnitude = CGFloat(slider.value)
    }
}
```

---

### Лучшие практики

1.  **Всегда добавляйте [[UICollisionBehavior]]:** Если не добавить столкновения, объекты будут падать бесконечно и исчезать за пределами экрана.
2.  **Комбинируйте с [[UIDynamicItemBehavior]]:** Для контроля упругости, трения и плотности всегда добавляйте это поведение.
3.  **Используйте `action` для сложных направлений:** Если вам нужна гравитация, меняющая направление в зависимости от положения (к центру, к пальцу), обновляйте `gravityDirection` в `action`.
4.  **Не создавайте слишком много элементов:** Гравитация на сотни объектов может снизить производительность. Используйте [[CADisplayLink]] для отладки FPS.

---

### Итог

**`UIGravityBehavior`** — это фундаментальный строительный блок для любой физической анимации в UIKit. Он позволяет:

1.  **Применять гравитацию** к любому количеству элементов.
2.  **Настраивать направление** (вниз, вверх, под углом, к центру).
3.  **Регулировать силу** притяжения.
4.  **Комбинироваться с другими поведениями** (столкновения, упругость, трение).
5.  **Динамически меняться** в рантайме через `action`.

Гравитация превращает статичный интерфейс в живую, физическую среду, где объекты падают, скользят и взаимодействуют естественно .