#uikit #uidynamics #collision #physics #ios #swift #optionset

---

## UICollisionBehaviorMode — Режимы столкновений

### Определение

**`UICollisionBehaviorMode`** — это опциональный тип (option set) в [[UIKit]], который определяет, **с чем** могут сталкиваться элементы, управляемые [[UICollisionBehavior]]. Он позволяет гибко настраивать, будут ли элементы сталкиваться друг с другом, с границами, или и с тем, и с другим .

Этот тип используется в свойстве `collisionMode` класса `UICollisionBehavior`.

### Зачем это знать iOS-разработчику?

1.  **Гибкая настройка:** Можно разрешить или запретить определённые типы столкновений в зависимости от сценария .
2.  **Оптимизация:** Отключение ненужных проверок столкновений может повысить производительность .
3.  **Специфичные эффекты:** Например, элементы могут отскакивать от стен, но проходить сквозь друг друга.
4.  **Игровая механика:** Создание "призрачных" объектов, которые не мешают друг другу, но реагируют на окружение.

---

### Возможные значения

`UICollisionBehaviorMode` является опциональным типом (`OptionSet`), поэтому значения можно комбинировать.

| Значение | Описание |
|---|---|
| **`.items`** | Элементы могут сталкиваться **друг с другом**. |
| **`.boundaries`** | Элементы могут сталкиваться с **границами** (добавленными через `addBoundary` или `translatesReferenceBoundsIntoBoundary`). |
| **`.everything`** | Комбинация `.items` и `.boundaries` (значение по умолчанию). Элементы сталкиваются и друг с другом, и с границами. |

#### Синтаксис

```swift
var collisionMode: UICollisionBehaviorMode
```

---

### Базовые примеры

#### 1. Только столкновения с границами (элементы проходят сквозь друг друга)

```swift
import UIKit

class BoundariesOnlyViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var collision: UICollisionBehavior!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        animator = UIDynamicAnimator(referenceView: view)
        
        // Создаем два шарика
        let ball1 = createBall(at: CGPoint(x: 100, y: 100), color: .systemRed)
        let ball2 = createBall(at: CGPoint(x: 150, y: 100), color: .systemBlue)
        
        // Гравитация для обоих
        let gravity = UIGravityBehavior(items: [ball1, ball2])
        animator.addBehavior(gravity)
        
        // Столкновения ТОЛЬКО с границами
        collision = UICollisionBehavior(items: [ball1, ball2])
        collision.translatesReferenceBoundsIntoBoundary = true
        collision.collisionMode = .boundaries  // ← Ключевая строка
        animator.addBehavior(collision)
        
        // Шарики будут падать и отскакивать от пола,
        // но будут проходить сквозь друг друга
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

#### 2. Только столкновения между элементами (игнорируют границы)

```swift
class ItemsOnlyViewController: UIViewController {
    var animator: UIDynamicAnimator!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        animator = UIDynamicAnimator(referenceView: view)
        
        let ball1 = createBall(at: CGPoint(x: 100, y: 100), color: .systemRed)
        let ball2 = createBall(at: CGPoint(x: 150, y: 100), color: .systemBlue)
        
        let gravity = UIGravityBehavior(items: [ball1, ball2])
        animator.addBehavior(gravity)
        
        let collision = UICollisionBehavior(items: [ball1, ball2])
        collision.collisionMode = .items  // ← Только друг с другом
        // Границы НЕ добавлены
        
        animator.addBehavior(collision)
        
        // Шарики будут сталкиваться друг с другом,
        // но будут падать сквозь границы экрана (и не остановятся)
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

#### 3. Все столкновения (значение по умолчанию)

```swift
class EverythingViewController: UIViewController {
    var animator: UIDynamicAnimator!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        animator = UIDynamicAnimator(referenceView: view)
        
        let ball1 = createBall(at: CGPoint(x: 100, y: 100), color: .systemRed)
        let ball2 = createBall(at: CGPoint(x: 150, y: 100), color: .systemBlue)
        
        let gravity = UIGravityBehavior(items: [ball1, ball2])
        animator.addBehavior(gravity)
        
        let collision = UICollisionBehavior(items: [ball1, ball2])
        collision.translatesReferenceBoundsIntoBoundary = true
        collision.collisionMode = .everything  // ← Значение по умолчанию (можно не указывать)
        animator.addBehavior(collision)
        
        // Шарики будут сталкиваться и друг с другом, и с границами экрана
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

### Комбинирование режимов (OptionSet)

Поскольку `UICollisionBehaviorMode` является `OptionSet`, можно комбинировать режимы вручную (хотя `.everything` уже делает это).

```swift
// Ручная комбинация (эквивалентно .everything)
collision.collisionMode = [.items, .boundaries]

// Проверка наличия режима
if collision.collisionMode.contains(.items) {
    print("Столкновения между элементами разрешены")
}
```

---

### Сложный пример: Игровая механика с переключением режимов

```swift
class GameViewController: UIViewController, UICollisionBehaviorDelegate {
    var animator: UIDynamicAnimator!
    var collision: UICollisionBehavior!
    var player: UIView!
    var enemies: [UIView] = []
    var isInvincible = false
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Создаем игрока
        player = UIView(frame: CGRect(x: view.bounds.midX - 25, y: view.bounds.height - 100, width: 50, height: 50))
        player.backgroundColor = .systemGreen
        player.layer.cornerRadius = 25
        view.addSubview(player)
        
        // Создаем врагов
        for i in 0..<3 {
            let enemy = UIView(frame: CGRect(x: 50 + i * 100, y: 100, width: 40, height: 40))
            enemy.backgroundColor = .systemRed
            enemy.layer.cornerRadius = 20
            view.addSubview(enemy)
            enemies.append(enemy)
        }
        
        animator = UIDynamicAnimator(referenceView: view)
        
        // Гравитация для всех
        let gravity = UIGravityBehavior(items: [player] + enemies)
        gravity.gravityDirection = CGVector(dx: 0, dy: 0.5)
        animator.addBehavior(gravity)
        
        // Столкновения
        collision = UICollisionBehavior(items: [player] + enemies)
        collision.translatesReferenceBoundsIntoBoundary = true
        collision.collisionDelegate = self
        collision.collisionMode = .everything  // По умолчанию
        animator.addBehavior(collision)
        
        // Жест для управления игроком
        let pan = UIPanGestureRecognizer(target: self, action: #selector(handlePan))
        view.addGestureRecognizer(pan)
    }
    
    @objc func handlePan(_ gesture: UIPanGestureRecognizer) {
        let location = gesture.location(in: view)
        player.center = CGPoint(x: location.x, y: player.center.y)
        animator.updateItemUsingCurrentState(player)
    }
    
    func makeInvincible(for duration: TimeInterval) {
        isInvincible = true
        player.backgroundColor = .systemYellow
        
        // Временно отключаем столкновения с врагами
        collision.collisionMode = .boundaries
        
        DispatchQueue.main.asyncAfter(deadline: .now() + duration) { [weak self] in
            self?.isInvincible = false
            self?.player.backgroundColor = .systemGreen
            self?.collision.collisionMode = .everything
        }
    }
    
    // MARK: - UICollisionBehaviorDelegate
    
    func collisionBehavior(_ behavior: UICollisionBehavior,
                          beganContactFor item1: UIDynamicItem,
                          with item2: UIDynamicItem,
                          at p: CGPoint) {
        
        if !isInvincible {
            if (item1 as? UIView == player && enemies.contains(item2 as? UIView)) ||
               (item2 as? UIView == player && enemies.contains(item1 as? UIView)) {
                print("💀 Игрок столкнулся с врагом!")
                makeInvincible(for: 2.0)
            }
        }
    }
}
```

---

### Сценарии использования различных режимов

| Режим | Сценарий |
|---|---|
| **`.everything`** | Стандартная физика (бильярд, пинбол, падающие объекты). |
| **`.boundaries`** | Частицы, которые не должны взаимодействовать друг с другом, но должны удерживаться в области (например, дождь, снег, конфетти). |
| **`.items`** | Объекты, которые могут сталкиваться друг с другом, но не имеют границ (например, астероиды в открытом космосе). |

---

### Лучшие практики

1.  **По умолчанию используйте `.everything`:** Это наиболее естественное поведение для большинства физических сценариев.
2.  **Отключайте ненужные проверки для производительности:** Если вам не нужны столкновения между элементами или с границами, отключите их.
3.  **Комбинируйте с делегатом:** Даже если столкновения между элементами отключены (`.boundaries`), делегат всё равно будет вызываться для столкновений с границами.
4.  **Динамически меняйте режимы:** В играх можно временно отключать столкновения (например, режим неуязвимости).

```swift
// Пример динамического переключения
func enableGhostMode() {
    collision.collisionMode = .boundaries  // Проходит сквозь врагов
}

func disableGhostMode() {
    collision.collisionMode = .everything
}
```

---

### Итог

**`UICollisionBehaviorMode`** — это простой, но мощный инструмент для настройки физических взаимодействий в [[UIKit]] Dynamics. Он позволяет:

1.  **Разрешать или запрещать** столкновения между элементами и с границами.
2.  **Оптимизировать производительность** отключением ненужных проверок.
3.  **Создавать разнообразные физические эффекты** — от реалистичных симуляций до "призрачных" объектов.
4.  **Динамически менять поведение** в рантайме (например, режим неуязвимости в игре).

Понимание этого типа необходимо для точной настройки `UICollisionBehavior` в любом проекте, использующем физическую анимацию .