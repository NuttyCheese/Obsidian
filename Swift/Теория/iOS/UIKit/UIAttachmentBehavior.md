#uikit #uidynamics #attachment #spring #physics #ios #swift

---

## UIAttachmentBehavior — Эффект "резинки" и связывание объектов

### Определение

**`UIAttachmentBehavior`** — это класс в [[UIKit]], наследующий от [[UIDynamicBehavior]]. Он создает физическую связь ("присоску" или "пружину") между элементом и точкой, либо между двумя элементами. Эта связь заставляет элемент(ы) двигаться так, как будто они привязаны резинкой или пружиной к чему-то .

Это поведение идеально подходит для создания эффектов перетаскивания с "инерцией" (drag & drop), окон, которые "прилипают" к краям, или любых интерфейсных элементов, которые должны следовать за пальцем с задержкой и отскоком.

### Зачем это знать iOS-разработчику?

1.  **Эффект "резинки" (spring effect):** Создает плавное, физически правдоподобное движение с затухающими колебаниями, которое сложно воспроизвести стандартными анимациями .
2.  **Связывание объектов:** Позволяет "привязать" один элемент к другому (например, иконку к окну или платформу к персонажу в игре) .
3.  **Интерактивность:** Идеально сочетается с жестами перетаскивания ([[UIPanGestureRecognizer]]), позволяя элементу "следовать" за пальцем с эффектом задержки .
4.  **Тонкая настройка:** Параметры `frequency` (частота) и `damping` (демпфирование) дают полный контроль над поведением "пружины".

---

### Архитектура и ключевые свойства

#### Типы инициализации

`UIAttachmentBehavior` имеет три основных варианта создания:

```swift
// 1. Привязка элемента к якорной точке
init(item: UIDynamicItem, attachedToAnchor: CGPoint)

// 2. Привязка элемента к якорной точке с указанием смещения (offset)
init(item: UIDynamicItem, offsetFromCenter: UIOffset, attachedToAnchor: CGPoint)

// 3. Привязка двух элементов друг к другу
init(item: UIDynamicItem, attachedTo: UIDynamicItem)

// 4. Привязка двух элементов с указанием смещений для обоих
init(item: UIDynamicItem, offsetFromCenter: UIOffset, attachedTo: UIDynamicItem, offsetFromCenter: UIOffset)
```

#### Основные свойства

| Свойство          | Тип             | Описание                                                                                                                                                                |
| ----------------- | --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`length`**      | `CGFloat`       | Длина "резинки" в точках. Расстояние, на котором связь находится в состоянии покоя. По умолчанию: `0`.                                                                  |
| **`frequency`**   | `CGFloat`       | Частота колебаний пружины. Чем выше значение, тем жестче связь (быстрее возврат в исходное положение). `0.0` означает, что связь не колеблется (обычная, не пружинная). |
| **`damping`**     | [[CGFloat]]     | Демпфирование (затухание) колебаний. Чем выше значение, тем быстрее затухают колебания. Диапазон: `0.0` (бесконечные колебания) до `1.0` (мгновенная остановка).        |
| **`anchorPoint`** | [[CGPoint]]     | Точка привязки (для типа "элемент-точка"). Может изменяться динамически.                                                                                                |
| **`action`**      | `(() -> Void)?` | Блок кода, вызываемый в каждом кадре анимации (полезно для обновления UI).                                                                                              |

> **Важно:** Если `frequency` равна `0`, поведение становится **непружинным** (обычное присоединение без колебаний). Если `frequency` больше `0`, включается пружинный эффект.

---

### Базовый пример: Эффект "резинки" для кнопки

Создадим кнопку, которая следует за пальцем с эффектом задержки и отскока.

```swift
import UIKit

class AttachmentViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var attachment: UIAttachmentBehavior?
    var button: UIButton!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        
        // Создаем кнопку
        button = UIButton(frame: CGRect(x: 100, y: 200, width: 100, height: 100))
        button.backgroundColor = .systemBlue
        button.layer.cornerRadius = 50
        button.addTarget(self, action: #selector(handlePan(_:)), for: .touchDragInside)
        view.addSubview(button)
        
        // Инициализируем аниматор
        animator = UIDynamicAnimator(referenceView: view)
    }
    
    @objc func handlePan(_ gesture: UIPanGestureRecognizer) {
        let location = gesture.location(in: view)
        
        switch gesture.state {
        case .began:
            // Создаем "присоску" между кнопкой и точкой касания
            attachment = UIAttachmentBehavior(item: button, attachedToAnchor: location)
            attachment?.frequency = 2.0      // Высокая частота (жесткая пружина)
            attachment?.damping = 0.5        // Среднее затухание
            animator.addBehavior(attachment!)
            
        case .changed:
            // Обновляем точку привязки
            attachment?.anchorPoint = location
            
        case .ended, .cancelled:
            // Удаляем поведение, когда палец отпущен
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

### Продвинутый пример: Связь двух элементов

Создадим квадрат, который привязан к кругу резинкой, и заставим круг следовать за пальцем.

```swift
class TwoElementsAttachmentViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var attachment: UIAttachmentBehavior?
    var circle: UIView!
    var square: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        
        // Создаем круг (будет двигаться за пальцем)
        circle = UIView(frame: CGRect(x: 150, y: 300, width: 60, height: 60))
        circle.backgroundColor = .systemRed
        circle.layer.cornerRadius = 30
        view.addSubview(circle)
        
        // Создаем квадрат (будет следовать за кругом на резинке)
        square = UIView(frame: CGRect(x: 100, y: 400, width: 50, height: 50))
        square.backgroundColor = .systemGreen
        square.layer.cornerRadius = 8
        view.addSubview(square)
        
        animator = UIDynamicAnimator(referenceView: view)
        
        // Добавляем гравитацию для квадрата (он будет "падать", если оторвется)
        let gravity = UIGravityBehavior(items: [square])
        gravity.gravityDirection = CGVector(dx: 0, dy: 0.8)
        animator.addBehavior(gravity)
        
        // Добавляем столкновения для границ экрана
        let collision = UICollisionBehavior(items: [circle, square])
        collision.translatesReferenceBoundsIntoBoundary = true
        animator.addBehavior(collision)
        
        // Связываем круг и квадрат пружиной
        attachment = UIAttachmentBehavior(item: circle, attachedTo: square)
        attachment?.length = 80                      // Расстояние покоя
        attachment?.frequency = 1.5                  // Частота колебаний
        attachment?.damping = 0.3                    // Слабое затухание (будет долго "трястись")
        animator.addBehavior(attachment!)
        
        // Добавляем жест перетаскивания для круга
        let pan = UIPanGestureRecognizer(target: self, action: #selector(handlePan))
        circle.addGestureRecognizer(pan)
        circle.isUserInteractionEnabled = true
    }
    
    @objc func handlePan(_ gesture: UIPanGestureRecognizer) {
        let location = gesture.location(in: view)
        
        switch gesture.state {
        case .changed:
            // Перемещаем круг, квадрат будет "тянуться" за ним на резинке
            circle.center = location
            // Обновляем состояние в аниматоре
            animator.updateItemUsingCurrentState(circle)
            
        default: break
        }
    }
}
```

---

### Смещение центра (Offset)

По умолчанию присоединение происходит к центру элемента. Используя `offsetFromCenter`, можно привязать элемент за угол или любую другую точку.

```swift
class OffsetAttachmentViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var attachment: UIAttachmentBehavior?
    var viewToDrag: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        viewToDrag = UIView(frame: CGRect(x: 100, y: 200, width: 120, height: 120))
        viewToDrag.backgroundColor = .systemPurple
        viewToDrag.layer.cornerRadius = 12
        view.addSubview(viewToDrag)
        
        animator = UIDynamicAnimator(referenceView: view)
        
        // Жест для перетаскивания
        let pan = UIPanGestureRecognizer(target: self, action: #selector(handlePan))
        viewToDrag.addGestureRecognizer(pan)
        viewToDrag.isUserInteractionEnabled = true
    }
    
    @objc func handlePan(_ gesture: UIPanGestureRecognizer) {
        let location = gesture.location(in: view)
        
        switch gesture.state {
        case .began:
            // Вычисляем смещение от центра до точки касания
            let touchPoint = gesture.location(in: viewToDrag)
            let offset = UIOffset(
                horizontal: touchPoint.x - viewToDrag.bounds.width / 2,
                vertical: touchPoint.y - viewToDrag.bounds.height / 2
            )
            
            // Привязываем элемент за точку касания, а не за центр
            attachment = UIAttachmentBehavior(
                item: viewToDrag,
                offsetFromCenter: offset,
                attachedToAnchor: location
            )
            attachment?.frequency = 2.5
            attachment?.damping = 0.6
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

### Комбинирование с другими поведениями

[[UIAttachmentBehavior]] отлично работает в связке с другими физическими силами.

```swift
class CombinedBehaviorsViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var dynamicItem: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        dynamicItem = UIView(frame: CGRect(x: 150, y: 100, width: 80, height: 80))
        dynamicItem.backgroundColor = .systemTeal
        dynamicItem.layer.cornerRadius = 40
        view.addSubview(dynamicItem)
        
        animator = UIDynamicAnimator(referenceView: view)
        
        // 1. Гравитация (тянет вниз)
        let gravity = UIGravityBehavior(items: [dynamicItem])
        gravity.gravityDirection = CGVector(dx: 0, dy: 0.8)
        animator.addBehavior(gravity)
        
        // 2. Столкновения (с границами экрана)
        let collision = UICollisionBehavior(items: [dynamicItem])
        collision.translatesReferenceBoundsIntoBoundary = true
        animator.addBehavior(collision)
        
        // 3. Присоединение к точке (будет "тянуть" вверх)
        let attachment = UIAttachmentBehavior(item: dynamicItem, attachedToAnchor: CGPoint(x: view.center.x, y: 150))
        attachment.length = 100
        attachment.frequency = 1.2
        attachment.damping = 0.4
        animator.addBehavior(attachment)
    }
}
```

---

### Решение проблемы: Обновление состояния элемента

Если вы вручную меняете положение элемента (например, `center`), аниматор не знает об этом и может "сбросить" ваши изменения.

```swift
// Неправильно: аниматор не в курсе изменений
myView.center = newCenter

// Правильно: сообщаем аниматору об изменении
myView.center = newCenter
animator.updateItemUsingCurrentState(myView)
```

Это особенно важно при комбинировании с `UIAttachmentBehavior` и ручным управлением.

---

### Лучшие практики

1.  **Экспериментируйте с параметрами:** `frequency` и `damping` дают огромный диапазон эффектов:
    - Высокая частота + высокое демпфирование = жесткая, быстро затухающая пружина.
    - Низкая частота + низкое демпфирование = мягкая, долго "трясущаяся" резинка.
2.  **Удаляйте поведения:** Всегда удаляйте `UIAttachmentBehavior` из аниматора, когда он больше не нужен (например, после окончания жеста), чтобы освободить ресурсы .
3.  **Не создавайте длинных связей:** Для очень длинных расстояний между элементами лучше использовать [[UISnapBehavior]], так как длинная "резинка" выглядит неестественно .
4.  **Используйте `action` для обратной связи:** Свойство `action` позволяет выполнять код в каждом кадре анимации.

```swift
attachment.action = {
    if self.square.center.y > 600 {
        print("Квадрат упал слишком низко!")
    }
}
```

---

### Итог

**`UIAttachmentBehavior`** — это гибкий и мощный инструмент для создания физических связей в интерфейсе. Он позволяет реализовать эффект "резинки", "присоски" или "пружины", делая анимации более живыми и интерактивными. Понимание параметров `length`, `frequency` и `damping` дает полный контроль над поведением, позволяя создавать как едва заметные эффекты, так и яркие игровые механики .