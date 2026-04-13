#uikit #uidynamics #collision #delegate #physics #ios #swift

---

## UICollisionBehaviorDelegate — Отслеживание столкновений

### Определение

**`UICollisionBehaviorDelegate`** — это протокол в [[UIKit]], который позволяет отслеживать **моменты начала и окончания столкновений** между элементами и между элементами и границами. Делегат вызывается автоматически [[UICollisionBehavior]] при каждом контакте .

Этот механизм не влияет на физику столкновений, но даёт возможность реагировать на них: воспроизводить звуки, вибрацию, обновлять UI, логировать события или запускать дополнительные анимации.

### Зачем это знать iOS-разработчику?

1.  **Обратная связь:** Воспроизведение звука или вибрации при ударе (например, в игре-бильярд) .
2.  **Игровая механика:** Начисление очков при попадании в цель, удаление объектов при столкновении.
3.  **Обновление UI:** Изменение цвета элемента при ударе, подсветка границ .
4.  **Отладка:** Логирование контактов для настройки физических параметров .
5.  **События в интерфейсе:** Триггер действий при достижении объектом определенной зоны.

---

### Методы протокола

`UICollisionBehaviorDelegate` предоставляет четыре метода (все опциональные):

#### 1. Столкновение элемента с границей (начало)

```swift
optional func collisionBehavior(
    _ behavior: UICollisionBehavior,
    beganContactFor item: UIDynamicItem,
    withBoundaryIdentifier identifier: NSCopying?,
    at p: CGPoint
)
```

- **`behavior`** — экземпляр `UICollisionBehavior`, который вызвал событие.
- **`item`** — элемент, участвующий в столкновении.
- **`identifier`** — идентификатор границы (если был задан при добавлении).
- **`p`** — точка контакта в координатах reference view.

#### 2. Столкновение элемента с границей (конец)

```swift
optional func collisionBehavior(
    _ behavior: UICollisionBehavior,
    endedContactFor item: UIDynamicItem,
    withBoundaryIdentifier identifier: NSCopying?
)
```

#### 3. Столкновение двух элементов (начало)

```swift
optional func collisionBehavior(
    _ behavior: UICollisionBehavior,
    beganContactFor item1: UIDynamicItem,
    with item2: UIDynamicItem,
    at p: CGPoint
)
```

#### 4. Столкновение двух элементов (конец)

```swift
optional func collisionBehavior(
    _ behavior: UICollisionBehavior,
    endedContactFor item1: UIDynamicItem,
    with item2: UIDynamicItem
)
```

---

### Базовый пример: Логирование столкновений

```swift
import UIKit

class CollisionLoggerViewController: UIViewController, UICollisionBehaviorDelegate {
    var animator: UIDynamicAnimator!
    var collision: UICollisionBehavior!
    var ball: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        ball = UIView(frame: CGRect(x: 100, y: 100, width: 50, height: 50))
        ball.backgroundColor = .systemBlue
        ball.layer.cornerRadius = 25
        view.addSubview(ball)
        
        animator = UIDynamicAnimator(referenceView: view)
        
        let gravity = UIGravityBehavior(items: [ball])
        animator.addBehavior(gravity)
        
        collision = UICollisionBehavior(items: [ball])
        collision.translatesReferenceBoundsIntoBoundary = true
        collision.collisionDelegate = self
        animator.addBehavior(collision)
    }
    
    // MARK: - UICollisionBehaviorDelegate
    
    func collisionBehavior(_ behavior: UICollisionBehavior,
                          beganContactFor item: UIDynamicItem,
                          withBoundaryIdentifier identifier: NSCopying?,
                          at p: CGPoint) {
        print("🔵 Шар ударился о границу \(identifier ?? "unknown") в точке \(p)")
    }
    
    func collisionBehavior(_ behavior: UICollisionBehavior,
                          beganContactFor item1: UIDynamicItem,
                          with item2: UIDynamicItem,
                          at p: CGPoint) {
        print("🟢 Шар столкнулся с другим объектом в точке \(p)")
    }
}
```

---

### Продвинутый пример: Звуки и вибрация

```swift
import UIKit
import AVFoundation

class CollisionFeedbackViewController: UIViewController, UICollisionBehaviorDelegate {
    var animator: UIDynamicAnimator!
    var collision: UICollisionBehavior!
    var balls: [UIView] = []
    var audioPlayer: AVAudioPlayer?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupAudio()
        
        // Добавляем несколько шариков
        for i in 0..<5 {
            addBall(at: CGPoint(x: 50 + i * 70, y: 100))
        }
        
        animator = UIDynamicAnimator(referenceView: view)
        
        let gravity = UIGravityBehavior(items: balls)
        animator.addBehavior(gravity)
        
        collision = UICollisionBehavior(items: balls)
        collision.translatesReferenceBoundsIntoBoundary = true
        collision.collisionDelegate = self
        animator.addBehavior(collision)
    }
    
    func addBall(at point: CGPoint) {
        let ball = UIView(frame: CGRect(x: point.x, y: point.y, width: 50, height: 50))
        ball.backgroundColor = .systemBlue
        ball.layer.cornerRadius = 25
        view.addSubview(ball)
        balls.append(ball)
    }
    
    func setupAudio() {
        guard let url = Bundle.main.url(forResource: "click", withExtension: "wav") else { return }
        audioPlayer = try? AVAudioPlayer(contentsOf: url)
        audioPlayer?.prepareToPlay()
    }
    
    // MARK: - UICollisionBehaviorDelegate
    
    func collisionBehavior(_ behavior: UICollisionBehavior,
                          beganContactFor item: UIDynamicItem,
                          withBoundaryIdentifier identifier: NSCopying?,
                          at p: CGPoint) {
        // Воспроизводим звук при ударе о стену
        audioPlayer?.play()
        
        // Визуальная обратная связь
        if let view = item as? UIView {
            UIView.animate(withDuration: 0.1) {
                view.backgroundColor = .systemRed
            } completion: { _ in
                UIView.animate(withDuration: 0.1) {
                    view.backgroundColor = .systemBlue
                }
            }
        }
    }
    
    func collisionBehavior(_ behavior: UICollisionBehavior,
                          beganContactFor item1: UIDynamicItem,
                          with item2: UIDynamicItem,
                          at p: CGPoint) {
        // Вибрация при столкновении шариков
        let generator = UIImpactFeedbackGenerator(style: .light)
        generator.impactOccurred()
        
        // Визуальная обратная связь для обоих
        if let view1 = item1 as? UIView, let view2 = item2 as? UIView {
            view1.backgroundColor = .systemOrange
            view2.backgroundColor = .systemOrange
            
            DispatchQueue.main.asyncAfter(deadline: .now() + 0.1) {
                view1.backgroundColor = .systemBlue
                view2.backgroundColor = .systemBlue
            }
        }
    }
}
```

---

### Игровой сценарий: Уничтожение при столкновении

```swift
class GameCollisionViewController: UIViewController, UICollisionBehaviorDelegate {
    var animator: UIDynamicAnimator!
    var collision: UICollisionBehavior!
    var target: UIView!
    var projectiles: [UIView] = []
    var score = 0
    
    @IBOutlet weak var scoreLabel: UILabel!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Создаем цель
        target = UIView(frame: CGRect(x: view.bounds.midX - 40, y: 200, width: 80, height: 80))
        target.backgroundColor = .systemGreen
        target.layer.cornerRadius = 40
        view.addSubview(target)
        
        animator = UIDynamicAnimator(referenceView: view)
        
        collision = UICollisionBehavior(items: [target])
        collision.collisionDelegate = self
        animator.addBehavior(collision)
        
        // Добавляем кнопку для запуска снаряда
        let fireButton = UIButton(frame: CGRect(x: 50, y: view.bounds.height - 100, width: 100, height: 50))
        fireButton.setTitle("Огонь", for: .normal)
        fireButton.backgroundColor = .systemRed
        fireButton.addTarget(self, action: #selector(fireProjectile), for: .touchUpInside)
        view.addSubview(fireButton)
    }
    
    @objc func fireProjectile() {
        let projectile = UIView(frame: CGRect(x: 50, y: 400, width: 30, height: 30))
        projectile.backgroundColor = .systemOrange
        projectile.layer.cornerRadius = 15
        view.addSubview(projectile)
        projectiles.append(projectile)
        
        // Добавляем в collision
        collision.addItem(projectile)
        
        // Толчок для движения
        let push = UIPushBehavior(items: [projectile], mode: .instantaneous)
        push.angle = 0 // Вправо
        push.magnitude = 1.5
        animator.addBehavior(push)
        
        // Таймер для удаления снаряда через 3 секунды
        DispatchQueue.main.asyncAfter(deadline: .now() + 3) { [weak self] in
            self?.removeProjectile(projectile)
        }
    }
    
    func removeProjectile(_ projectile: UIView) {
        collision.removeItem(projectile)
        projectile.removeFromSuperview()
        projectiles.removeAll { $0 == projectile }
    }
    
    // MARK: - UICollisionBehaviorDelegate
    
    func collisionBehavior(_ behavior: UICollisionBehavior,
                          beganContactFor item1: UIDynamicItem,
                          with item2: UIDynamicItem,
                          at p: CGPoint) {
        // Определяем, что снаряд попал в цель
        if let projectile = item1 as? UIView, projectile != target,
           let targetHit = item2 as? UIView, targetHit == target {
            handleHit(projectile: projectile)
        } else if let projectile = item2 as? UIView, projectile != target,
                  let targetHit = item1 as? UIView, targetHit == target {
            handleHit(projectile: projectile)
        }
    }
    
    func handleHit(projectile: UIView) {
        score += 10
        scoreLabel.text = "Score: \(score)"
        
        // Визуальный эффект попадания
        target.backgroundColor = .systemYellow
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.1) {
            self.target.backgroundColor = .systemGreen
        }
        
        // Удаляем снаряд
        removeProjectile(projectile)
        
        // Вибрация
        let generator = UIImpactFeedbackGenerator(style: .heavy)
        generator.impactOccurred()
    }
}
```

---

### Отслеживание границ с идентификаторами

```swift
class BoundaryTrackingViewController: UIViewController, UICollisionBehaviorDelegate {
    var animator: UIDynamicAnimator!
    var collision: UICollisionBehavior!
    var ball: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        ball = UIView(frame: CGRect(x: 100, y: 100, width: 40, height: 40))
        ball.backgroundColor = .systemPurple
        ball.layer.cornerRadius = 20
        view.addSubview(ball)
        
        animator = UIDynamicAnimator(referenceView: view)
        
        let gravity = UIGravityBehavior(items: [ball])
        animator.addBehavior(gravity)
        
        collision = UICollisionBehavior(items: [ball])
        collision.collisionDelegate = self
        
        // Добавляем границы с идентификаторами
        collision.addBoundary(withIdentifier: "top" as NSCopying,
                              from: CGPoint(x: 0, y: 100),
                              to: CGPoint(x: view.bounds.width, y: 100))
        
        collision.addBoundary(withIdentifier: "bottom" as NSCopying,
                              from: CGPoint(x: 0, y: view.bounds.height - 100),
                              to: CGPoint(x: view.bounds.width, y: view.bounds.height - 100))
        
        animator.addBehavior(collision)
    }
    
    // MARK: - UICollisionBehaviorDelegate
    
    func collisionBehavior(_ behavior: UICollisionBehavior,
                          beganContactFor item: UIDynamicItem,
                          withBoundaryIdentifier identifier: NSCopying?,
                          at p: CGPoint) {
        
        guard let id = identifier as? String else { return }
        
        switch id {
        case "top":
            print("📈 Шар коснулся верхней границы")
            // Изменяем цвет при касании верха
            (item as? UIView)?.backgroundColor = .systemRed
            
        case "bottom":
            print("📉 Шар коснулся нижней границы")
            (item as? UIView)?.backgroundColor = .systemGreen
            
        default:
            print("🟡 Шар коснулся другой границы")
        }
        
        // Возвращаем цвет через 0.2 секунды
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.2) { [weak self] in
            (item as? UIView)?.backgroundColor = self?.ball.backgroundColor
        }
    }
}
```

---

### Лучшие практики

1.  **Проверяйте идентификаторы:** При использовании множества границ всегда проверяйте `identifier`, чтобы понять, с чем именно произошло столкновение.
2.  **Избегайте тяжелых операций в делегате:** Методы делегата вызываются в каждом кадре при активных контактах. Не выполняйте синхронных дисковых операций или сложных вычислений.
3.  **Удаляйте элементы из поведений:** При удалении [[UIView]] из иерархии не забудьте удалить его из `UICollisionBehavior`, иначе делегат будет продолжать получать события для удаленного объекта.
4.  **Используйте `endedContactFor` для сброса состояния:** Если вы меняете цвет или другие свойства при начале контакта, верните их обратно при окончании.

```swift
func collisionBehavior(_ behavior: UICollisionBehavior,
                      endedContactFor item: UIDynamicItem,
                      withBoundaryIdentifier identifier: NSCopying?) {
    if let view = item as? UIView {
        view.backgroundColor = .systemBlue
    }
}
```

5.  **Помните о множественных контактах:** Один элемент может одновременно контактировать с несколькими границами или другими элементами. Делегат будет вызван для каждого контакта.

---

### Итог

**`UICollisionBehaviorDelegate`** — это мощный инструмент для обратной связи и игровой механики в UIKit Dynamics. Он позволяет:

1.  **Реагировать на столкновения** между элементами и с границами.
2.  **Добавлять звуковые и тактильные эффекты** при ударах.
3.  **Реализовывать игровую логику** (начисление очков, уничтожение объектов).
4.  **Обновлять UI** в ответ на физические взаимодействия.
5.  **Отслеживать точку контакта** для точных расчетов.

Использование делегата превращает простую физическую анимацию в интерактивный, отзывчивый интерфейс .