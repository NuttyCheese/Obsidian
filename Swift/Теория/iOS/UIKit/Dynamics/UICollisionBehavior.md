#uikit #uidynamics #collision #physics #ios #swift

---

## UICollisionBehavior — Столкновения и границы в физической анимации

### Определение

**`UICollisionBehavior`** — это класс в [[UIKit]], наследующий от [[UIDynamicBehavior]]. Он добавляет элементам возможность **сталкиваться** друг с другом и с заданными границами (boundaries). Без этого поведения объекты будут проходить сквозь друг друга и сквозь границы экрана .

`UICollisionBehavior` работает в паре с [[UIDynamicAnimator]] и обычно используется вместе с [[UIGravityBehavior]], [[UIPushBehavior]] или [[UIAttachmentBehavior]], чтобы создать реалистичную физическую среду.

### Зачем это знать iOS-разработчику?

1.  **Реалистичные столкновения:** Объекты могут отталкиваться друг от друга, падать на пол, ударяться о стены .
2.  **Игровая механика:** Создание платформеров, бильярда, пинбола и других игр с физикой.
3.  **Интерактивные интерфейсы:** Эффекты "падающих" элементов меню, карточек, которые сталкиваются друг с другом .
4.  **Границы экрана:** Легко ограничить движение объектов пределами reference view или произвольными линиями.
5.  **Обратная связь:** Через делегат можно отслеживать момент столкновения и воспроизводить звуки или вибрацию.

---

### Архитектура и ключевые свойства

#### Инициализация

```swift
init(items: [UIDynamicItem])
```

#### Основные свойства и методы

| Свойство / Метод                            | Тип                             | Описание                                                                                                                                                                                       |
| ------------------------------------------- | ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`translatesReferenceBoundsIntoBoundary`** | [[Bool]]                        | Если `true`, границы reference view (обычно `view`) становятся непреодолимой стеной для элементов.                                                                                             |
| **`collisionMode`**                         | [[UICollisionBehaviorMode]]     | Определяет, с чем объекты могут сталкиваться:<br>• `.items` — только друг с другом<br>• `.boundaries` — только с границами<br>• `.everything` — и с границами, и друг с другом (по умолчанию). |
| **`addBoundary(withIdentifier:from:to:)`**  | Метод                           | Добавляет линейную границу (стену) с указанным идентификатором.                                                                                                                                |
| **`addBoundary(withIdentifier:for:)`**      | Метод                           | Добавляет границу в виде UIBezierPath (можно создавать сложные формы).                                                                                                                         |
| **`removeBoundary(withIdentifier:)`**       | Метод                           | Удаляет границу по идентификатору.                                                                                                                                                             |
| **`removeAllBoundaries()`**                 | Метод                           | Удаляет все добавленные границы.                                                                                                                                                               |
| **`collisionDelegate`**                     | [[UICollisionBehaviorDelegate]] | Делегат для отслеживания событий столкновений.                                                                                                                                                 |

---

### Базовый пример: Падающие шарики и пол

```swift
import UIKit

class CollisionViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var gravity: UIGravityBehavior!
    var collision: UICollisionBehavior!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        
        // Создаем аниматор
        animator = UIDynamicAnimator(referenceView: view)
        
        // 1. Гравитация
        gravity = UIGravityBehavior(items: [])
        gravity.gravityDirection = CGVector(dx: 0, dy: 1.0) // Падение вниз
        animator.addBehavior(gravity)
        
        // 2. Столкновения
        collision = UICollisionBehavior(items: [])
        collision.translatesReferenceBoundsIntoBoundary = true // Границы экрана
        animator.addBehavior(collision)
        
        // Добавляем несколько шариков
        for i in 0..<5 {
            addBall(at: CGPoint(x: 50 + i * 70, y: 100))
        }
    }
    
    func addBall(at point: CGPoint) {
        let size: CGFloat = 50
        let ball = UIView(frame: CGRect(x: point.x, y: point.y, width: size, height: size))
        ball.backgroundColor = .systemBlue
        ball.layer.cornerRadius = size / 2
        view.addSubview(ball)
        
        gravity.addItem(ball)
        collision.addItem(ball)
    }
}
```

---

### Кастомные границы: Стены, рамки и лабиринты

#### 1. **Линейная граница (стена)**

```swift
// Добавляем горизонтальную стену на высоте 400
collision.addBoundary(withIdentifier: "floor" as NSCopying,
                      from: CGPoint(x: 0, y: 400),
                      to: CGPoint(x: view.bounds.width, y: 400))
```

#### 2. **Прямоугольная рамка**

```swift
// Ограничиваем область прямоугольником
let rect = CGRect(x: 50, y: 150, width: view.bounds.width - 100, height: 300)
collision.addBoundary(withIdentifier: "boundingBox" as NSCopying,
                      for: UIBezierPath(rect: rect))
```

#### 3. **Круглая граница**

```swift
let center = CGPoint(x: view.bounds.midX, y: view.bounds.midY)
let radius: CGFloat = 150
let circlePath = UIBezierPath(arcCenter: center, radius: radius, startAngle: 0, endAngle: .pi * 2, clockwise: true)
collision.addBoundary(withIdentifier: "circle" as NSCopying, for: circlePath)
```

#### 4. **Полный пример: Лабиринт**

```swift
class MazeViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var gravity: UIGravityBehavior!
    var collision: UICollisionBehavior!
    var ball: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        animator = UIDynamicAnimator(referenceView: view)
        
        gravity = UIGravityBehavior(items: [])
        gravity.gravityDirection = CGVector(dx: 0.2, dy: 0.8) // Наклонный пол
        animator.addBehavior(gravity)
        
        collision = UICollisionBehavior(items: [])
        collision.translatesReferenceBoundsIntoBoundary = true
        animator.addBehavior(collision)
        
        // Создаем стены лабиринта
        addWall(from: CGPoint(x: 100, y: 200), to: CGPoint(x: 300, y: 200))
        addWall(from: CGPoint(x: 300, y: 200), to: CGPoint(x: 300, y: 400))
        addWall(from: CGPoint(x: 300, y: 400), to: CGPoint(x: 500, y: 400))
        addWall(from: CGPoint(x: 500, y: 400), to: CGPoint(x: 500, y: 600))
        
        // Добавляем шарик
        ball = UIView(frame: CGRect(x: 50, y: 50, width: 30, height: 30))
        ball.backgroundColor = .red
        ball.layer.cornerRadius = 15
        view.addSubview(ball)
        
        gravity.addItem(ball)
        collision.addItem(ball)
        
        // Добавляем гравитацию и столкновения для всех элементов
    }
    
    func addWall(from start: CGPoint, to end: CGPoint) {
        collision.addBoundary(withIdentifier: UUID().uuidString as NSCopying,
                              from: start, to: end)
        
        // Визуализация стены
        let wallView = UIView(frame: CGRect(x: start.x, y: start.y, width: hypot(end.x - start.x, end.y - start.y), height: 5))
        wallView.backgroundColor = .darkGray
        view.addSubview(wallView)
    }
}
```

---

### Режимы столкновений (`collisionMode`)

```swift
// Только столкновения между элементами (игнорируют границы)
collision.collisionMode = .items

// Только столкновения с границами (проходят сквозь друг друга)
collision.collisionMode = .boundaries

// И с границами, и друг с другом (по умолчанию)
collision.collisionMode = .everything
```

---

### Делегат: Отслеживание столкновений

`UICollisionBehaviorDelegate` позволяет узнать, когда произошло столкновение.

```swift
class CollisionDelegateViewController: UIViewController, UICollisionBehaviorDelegate {
    var animator: UIDynamicAnimator!
    var collision: UICollisionBehavior!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // ... настройка аниматора и collision ...
        
        collision.collisionDelegate = self
    }
    
    // Столкновение элемента с границей началось
    func collisionBehavior(_ behavior: UICollisionBehavior,
                          beganContactFor item: UIDynamicItem,
                          withBoundaryIdentifier identifier: NSCopying?,
                          at p: CGPoint) {
        print("Контакт с границей \(identifier ?? "unknown") в точке \(p)")
        
        // Визуальная обратная связь
        if let view = item as? UIView {
            view.backgroundColor = .yellow
            DispatchQueue.main.asyncAfter(deadline: .now() + 0.1) {
                view.backgroundColor = .systemBlue
            }
        }
    }
    
    // Столкновение элемента с границей закончилось
    func collisionBehavior(_ behavior: UICollisionBehavior,
                          endedContactFor item: UIDynamicItem,
                          withBoundaryIdentifier identifier: NSCopying?) {
        print("Контакт с границей \(identifier ?? "unknown") завершен")
    }
    
    // Столкновение двух элементов началось
    func collisionBehavior(_ behavior: UICollisionBehavior,
                          beganContactFor item1: UIDynamicItem,
                          with item2: UIDynamicItem,
                          at p: CGPoint) {
        print("Столкновение объектов в точке \(p)")
        
        // Эффект при столкновении
        if let view1 = item1 as? UIView, let view2 = item2 as? UIView {
            view1.backgroundColor = .orange
            view2.backgroundColor = .orange
        }
    }
    
    // Столкновение двух элементов закончилось
    func collisionBehavior(_ behavior: UICollisionBehavior,
                          endedContactFor item1: UIDynamicItem,
                          with item2: UIDynamicItem) {
        print("Объекты разошлись")
        
        if let view1 = item1 as? UIView, let view2 = item2 as? UIView {
            view1.backgroundColor = .systemBlue
            view2.backgroundColor = .systemBlue
        }
    }
}
```

---

### Комбинирование с другими поведениями

`UICollisionBehavior` редко используется в одиночку. Вот классическая связка:

```swift
class CompletePhysicsViewController: UIViewController {
    var animator: UIDynamicAnimator!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let box = UIView(frame: CGRect(x: 100, y: 100, width: 80, height: 80))
        box.backgroundColor = .systemGreen
        view.addSubview(box)
        
        animator = UIDynamicAnimator(referenceView: view)
        
        // 1. Гравитация
        let gravity = UIGravityBehavior(items: [box])
        gravity.gravityDirection = CGVector(dx: 0, dy: 1.0)
        animator.addBehavior(gravity)
        
        // 2. Столкновения (с границами экрана)
        let collision = UICollisionBehavior(items: [box])
        collision.translatesReferenceBoundsIntoBoundary = true
        animator.addBehavior(collision)
        
        // 3. Физические свойства (упругость, трение)
        let properties = UIDynamicItemBehavior(items: [box])
        properties.elasticity = 0.7  // Отскок при ударе
        properties.friction = 0.2    // Трение
        animator.addBehavior(properties)
        
        // 4. Толчок для запуска
        let push = UIPushBehavior(items: [box], mode: .instantaneous)
        push.angle = CGFloat.pi / 3  // 60 градусов
        push.magnitude = 2.0
        animator.addBehavior(push)
    }
}
```

---

### Решение проблемы: Элементы проваливаются сквозь границы

Если объекты проваливаются сквозь добавленные границы, проверьте:

1. **Порядок добавления:** Сначала добавьте границы, затем элементы (или убедитесь, что `collisionMode` не равен `.items`).
2. **Обновление состояния:** Если вы вручную меняете `center` элемента, вызовите `animator.updateItemUsingCurrentState(item)`.
3. **Толщина границы:** Линейные границы математически бесконечно тонкие. Для очень быстрых объектов может потребоваться увеличить `elasticity` или использовать `UISnapBehavior` для "притягивания".

```swift
// Если элемент "проскакивает" сквозь стену
let properties = UIDynamicItemBehavior(items: [item])
properties.elasticity = 0.5  // Увеличить упругость
properties.resistance = 0.5  // Увеличить сопротивление
```

---

### Лучшие практики

1.  **Используйте идентификаторы для границ:** При добавлении множества границ давайте им осмысленные идентификаторы — это облегчит отладку через делегат.
2.  **Не создавайте слишком много границ:** Сложные [[UIBezierPath]] с тысячами точек могут снизить производительность. Используйте простые линии и прямоугольники.
3.  **Комбинируйте с `UIDynamicItemBehavior`:** Для контроля упругости, трения и разрешения вращения при столкновениях всегда добавляйте [[UIDynamicItemBehavior]].
4.  **Удаляйте элементы из поведений:** При удалении [[UIView]] из иерархии обязательно удалите его из всех поведений, чтобы избежать утечек.

```swift
func removeBall(_ ball: UIView) {
    gravity.removeItem(ball)
    collision.removeItem(ball)
    ball.removeFromSuperview()
}
```

---

### Итог

**`UICollisionBehavior`** — это фундаментальный строительный блок для любой физической анимации в UIKit. Он позволяет элементам взаимодействовать друг с другом и с окружающей средой (стенами, полом, препятствиями). В сочетании с гравитацией, толчками и физическими свойствами он создает реалистичное поведение объектов, идеально подходящее как для игр, так и для нестандартных интерактивных интерфейсов .