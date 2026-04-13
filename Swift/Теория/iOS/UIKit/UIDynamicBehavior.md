#uikit #uidynamics #behavior #physics #ios #swift #composition

---

## UIDynamicBehavior — Базовый класс для физических правил

### Определение

**`UIDynamicBehavior`** — это абстрактный базовый класс в [[UIKit]], который определяет **правило** или **силу**, применяемую к элементам в [[UIDynamicAnimator]]. Сами по себе поведения ничего не анимируют — они лишь описывают физический закон (гравитацию, столкновение, прикрепление и т.д.). Вы добавляете поведения в аниматор, и тот просчитывает их совместное действие .

Все стандартные физические правила в UIKit Dynamics наследуются от `UIDynamicBehavior`:

- [[UIGravityBehavior]]
- [[UICollisionBehavior]]
- [[UIAttachmentBehavior]]
- [[UIPushBehavior]]
- [[UISnapBehavior]]
- [[UIDynamicItemBehavior]]

### Зачем это знать iOS-разработчику?

1.  **Создание кастомных поведений:** Вы можете создавать свои собственные поведения, комбинируя стандартные или добавляя уникальную логику .
2.  **Композиция:** Поведения можно группировать в дочерние поведения (`addChildBehavior`), создавая сложные физические правила из простых .
3.  **Переиспользование:** Группы поведений можно сохранять и применять к разным элементам или сценам.
4.  **Контроль жизненного цикла:** У каждого поведения есть методы, вызываемые при добавлении в аниматор и удалении из него.

---

### Архитектура и ключевые свойства

#### Базовый класс

```swift
class UIDynamicBehavior : NSObject
```

#### Основные свойства и методы

| Свойство / Метод | Описание |
|---|---|
| **`items`** | Массив элементов (`UIDynamicItem`), на которые влияет это поведение (только чтение). |
| **`action`** | Замыкание, вызываемое в каждом кадре анимации (полезно для отладки или обновления UI). |
| **`addChildBehavior(_:)`** | Добавляет дочернее поведение. Аниматор будет применять оба поведения. |
| **`removeChildBehavior(_:)`** | Удаляет дочернее поведение. |
| **`willMove(to:)`** | Вызывается перед добавлением поведения в аниматор (или `nil` при удалении). |
| **`dynamicAnimator`** | Аниматор, которому принадлежит поведение (weak ссылка). |

---

### Стандартные поведения (наследники)

Все стандартные поведения уже были подробно разобраны:

| Класс                         | Назначение                                                                         |
| ----------------------------- | ---------------------------------------------------------------------------------- |
| **[[UIGravityBehavior]]**     | Применяет силу гравитации.                                                         |
| **[[UICollisionBehavior]]**   | Добавляет столкновения с границами и другими элементами.                           |
| **[[UIAttachmentBehavior]]**  | Связывает элемент с точкой или другим элементом (резинка).                         |
| **[[UIPushBehavior]]**        | Прикладывает мгновенный или постоянный толчок.                                     |
| **[[UISnapBehavior]]**        | "Притягивает" элемент к точке с эффектом пружины.                                  |
| **[[UIDynamicItemBehavior]]** | Настраивает физические свойства элемента (трение, упругость, разрешение вращения). |

---

### Создание кастомного поведения

#### Пример 1: Поведение "Пульсация"

Создадим поведение, которое периодически увеличивает и уменьшает размер элемента.

```swift
class PulseBehavior: UIDynamicBehavior {
    private let item: UIDynamicItem
    private let originalTransform: CGAffineTransform
    private var displayLink: CADisplayLink?
    private var startTime: CFTimeInterval?
    
    init(item: UIDynamicItem) {
        self.item = item
        self.originalTransform = (item as? UIView)?.transform ?? .identity
        super.init()
        
        // Добавляем дочернее поведение для базовых физических свойств
        let itemBehavior = UIDynamicItemBehavior(items: [item])
        itemBehavior.allowsRotation = false
        addChildBehavior(itemBehavior)
    }
    
    override func willMove(to animator: UIDynamicAnimator?) {
        super.willMove(to: animator)
        
        if let animator = animator {
            // Добавлены в аниматор — запускаем пульсацию
            startPulsing()
        } else {
            // Удалены из аниматора — останавливаем
            stopPulsing()
            // Возвращаем исходный размер
            (item as? UIView)?.transform = originalTransform
        }
    }
    
    private func startPulsing() {
        displayLink = CADisplayLink(target: self, selector: #selector(updatePulse))
        displayLink?.add(to: .main, forMode: .common)
        startTime = CACurrentMediaTime()
    }
    
    private func stopPulsing() {
        displayLink?.invalidate()
        displayLink = nil
    }
    
    @objc private func updatePulse() {
        guard let startTime = startTime,
              let view = item as? UIView else { return }
        
        let elapsed = CACurrentMediaTime() - startTime
        let scale = 1 + sin(elapsed * 3) * 0.1 // Пульсация с частотой 3 Гц
        view.transform = originalTransform.scaledBy(x: scale, y: scale)
    }
}
```

#### Пример 2: Поведение "Следование за курсором"

```swift
class FollowMouseBehavior: UIDynamicBehavior {
    private let item: UIDynamicItem
    private weak var targetView: UIView?
    private var attachment: UIAttachmentBehavior?
    
    init(item: UIDynamicItem, in view: UIView) {
        self.item = item
        self.targetView = view
        super.init()
        
        // Добавляем физические свойства
        let itemBehavior = UIDynamicItemBehavior(items: [item])
        itemBehavior.resistance = 0.8 // Сопротивление движению
        addChildBehavior(itemBehavior)
    }
    
    func updateTargetPoint(_ point: CGPoint) {
        if attachment == nil {
            // Создаем "резинку" между элементом и точкой
            let newAttachment = UIAttachmentBehavior(item: item, attachedToAnchor: point)
            newAttachment.frequency = 2.0
            newAttachment.damping = 0.5
            newAttachment.length = 0
            addChildBehavior(newAttachment)
            attachment = newAttachment
        } else {
            attachment?.anchorPoint = point
        }
    }
    
    func detach() {
        if let attachment = attachment {
            removeChildBehavior(attachment)
            self.attachment = nil
        }
    }
}

// Использование
class MouseFollowViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var followBehavior: FollowMouseBehavior?
    var ball: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        ball = UIView(frame: CGRect(x: 100, y: 100, width: 60, height: 60))
        ball.backgroundColor = .systemRed
        ball.layer.cornerRadius = 30
        view.addSubview(ball)
        
        animator = UIDynamicAnimator(referenceView: view)
        followBehavior = FollowMouseBehavior(item: ball, in: view)
        animator.addBehavior(followBehavior!)
        
        let pan = UIPanGestureRecognizer(target: self, action: #selector(handlePan))
        view.addGestureRecognizer(pan)
    }
    
    @objc func handlePan(_ gesture: UIPanGestureRecognizer) {
        let point = gesture.location(in: view)
        
        switch gesture.state {
        case .changed:
            followBehavior?.updateTargetPoint(point)
        case .ended, .cancelled:
            followBehavior?.detach()
        default: break
        }
    }
}
```

---

### Композиция поведений (Child Behaviors)

Один из мощных паттернов — группировка нескольких поведений в одно кастомное.

```swift
class GravityWithCollisionBehavior: UIDynamicBehavior {
    private let gravity: UIGravityBehavior
    private let collision: UICollisionBehavior
    private let itemBehavior: UIDynamicItemBehavior
    
    init(items: [UIDynamicItem], in referenceView: UIView) {
        gravity = UIGravityBehavior(items: items)
        gravity.gravityDirection = CGVector(dx: 0, dy: 1.0)
        
        collision = UICollisionBehavior(items: items)
        collision.translatesReferenceBoundsIntoBoundary = true
        
        itemBehavior = UIDynamicItemBehavior(items: items)
        itemBehavior.elasticity = 0.6
        itemBehavior.friction = 0.3
        
        super.init()
        
        addChildBehavior(gravity)
        addChildBehavior(collision)
        addChildBehavior(itemBehavior)
    }
    
    func addItem(_ item: UIDynamicItem) {
        gravity.addItem(item)
        collision.addItem(item)
        itemBehavior.addItem(item)
    }
    
    func removeItem(_ item: UIDynamicItem) {
        gravity.removeItem(item)
        collision.removeItem(item)
        itemBehavior.removeItem(item)
    }
    
    var gravityDirection: CGVector {
        get { gravity.gravityDirection }
        set { gravity.gravityDirection = newValue }
    }
    
    var elasticity: CGFloat {
        get { itemBehavior.elasticity }
        set { itemBehavior.elasticity = newValue }
    }
}

// Использование
let physicsBehavior = GravityWithCollisionBehavior(items: [ball], in: view)
animator.addBehavior(physicsBehavior)
```

---

### Свойство `action`

`action` позволяет выполнять код в каждом кадре анимации.

```swift
let behavior = UIDynamicBehavior()
behavior.action = {
    print("Animation frame update")
    
    // Проверка условий, обновление UI, логирование
    if let ball = self.ball, ball.center.y > 500 {
        print("Ball fell below 500")
    }
}
animator.addBehavior(behavior)
```

---

### Жизненный цикл: `willMove(to:)`

Переопределяйте этот метод, чтобы выполнять настройку при добавлении/удалении поведения.

```swift
class LoggingBehavior: UIDynamicBehavior {
    let name: String
    
    init(name: String) {
        self.name = name
        super.init()
    }
    
    override func willMove(to animator: UIDynamicAnimator?) {
        if animator != nil {
            print("\(name) added to animator")
        } else {
            print("\(name) removed from animator")
        }
    }
}
```

---

### Лучшие практики

1.  **Композиция вместо наследования:** Вместо создания сложного монолитного поведения лучше скомбинировать несколько стандартных через `addChildBehavior`.
2.  **Управление памятью:** `UIDynamicAnimator` держит сильные ссылки на поведения. При удалении сцены обязательно удаляйте поведения или обнуляйте аниматор.
3.  **Используйте `action` для отладки:** Временно добавьте `action` в поведение, чтобы логировать позиции элементов и отлаживать физику.
4.  **Не перегружайте `willMove(to:)`:** Этот метод вызывается каждый раз при добавлении/удалении. Не выполняйте в нем тяжелых операций.
5.  **Проверяйте `dynamicAnimator`:** Если нужно узнать, активен ли аниматор, используйте свойство `dynamicAnimator` (оно будет [[nil]], если поведение не добавлено).

```swift
if behavior.dynamicAnimator != nil {
    // Поведение активно
}
```

---

### Итог

**`UIDynamicBehavior`** — это фундамент всей системы UIKit Dynamics. Понимание этого класса позволяет:

1.  **Создавать кастомные физические правила** для уникальных эффектов.
2.  **Группировать поведения** в переиспользуемые компоненты.
3.  **Управлять жизненным циклом** физических правил.
4.  **Отслеживать состояние анимации** через `action`.

Хотя в повседневной разработке вы чаще будете использовать готовые поведения (`UIGravityBehavior`, `UICollisionBehavior`), умение создавать свои собственные открывает двери к уникальным и сложным физическим взаимодействиям, недоступным из коробки .