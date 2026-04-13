#uikit #uidynamics #snap #animation #physics #ios #swift

---

## UISnapBehavior — Эффект "прилипания" с затуханием

### Определение

**`UISnapBehavior`** — это класс в [[UIKit]], наследующий от [[UIDynamicBehavior]]. Он представляет собой физический эффект, при котором объект (элемент) "перепрыгивает" в указанную точку с эффектом **затухающей пружины**. Объект не просто мгновенно телепортируется, а динамически двигается к цели, теряя скорость и амплитуду колебаний, пока не остановится .

Это поведение идеально подходит для создания меню, появляющихся при нажатии, возврата карточек в исходное положение после перетаскивания или любых интерфейсных элементов, которые должны "прилипать" к определенной позиции.

### Зачем это знать [[iOS]]-разработчику?

1.  **Реалистичная обратная связь:** Создает плавный, физически правдоподобный эффект "прилипания" или "втягивания", который сложно воспроизвести стандартными анимациями .
2.  **Простота использования:** В отличие от [[UIAttachmentBehavior]] (которое требует настройки точек привязки и частоты), `UISnapBehavior` требует всего два параметра: сам объект и целевую точку .
3.  **Интерактивность:** Идеально сочетается с жестами перетаскивания ([[UIPanGestureRecognizer]]), позволяя карточке "возвращаться на место" после того, как пользователь отпустил палец .
4.  **Эстетика интерфейсов:** Эффект пружины (`spring`) стал стандартом в iOS (например, в iOS 17+ это реализовано встроенными анимациями), и `UISnapBehavior` дает вам полный контроль над этим эффектом.

---

### Архитектура и ключевые свойства

#### Инициализация

```swift
init(item: UIDynamicItem, snapTo point: CGPoint)
```
- **`item`:** Анимируемый объект (обычно `UIView` или `UICollectionViewLayoutAttributes`).
- **`point`:** Целевая точка, в которую объект должен "прилипнуть". Координаты задаются относительно reference view аниматора.

#### Основные свойства

| Свойство                              | Тип                     | Описание                                                                                                                                                                                                                          |
| ------------------------------------- | ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`damping`**                         | `CGFloat`               | Уровень демпфирования (затухания).<br>• `0.0` → Колебания будут продолжаться очень долго (слабое затухание).<br>• `1.0` → Объект остановится максимально быстро, почти без колебаний (сильное затухание).<br>По умолчанию: `0.5`. |
| **`setDamping(_:forLinearDamping:)`** | ([[CGFloat]], [[Bool]]) | Уточнение затухания. Второй параметр отвечает за затухание линейной скорости (обычно оставляют `true`).                                                                                                                           |

> **Важно:** В отличие от `UIAttachmentBehavior`, здесь **нет** настройки `frequency` (частоты колебаний). Частота вычисляется автоматически на основе расстояния до целевой точки.

---

### Базовый пример: Анимированное меню

```swift
import UIKit

class SnapMenuViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var snapBehavior: UISnapBehavior?

    override func viewDidLoad() {
        super.viewDidLoad()
        animator = UIDynamicAnimator(referenceView: view)
        
        // Создаем кнопку в левом верхнем углу
        let menuButton = UIButton(frame: CGRect(x: 20, y: 100, width: 60, height: 60))
        menuButton.backgroundColor = .systemBlue
        menuButton.layer.cornerRadius = 30
        menuButton.addTarget(self, action: #selector(toggleMenu), for: .touchUpInside)
        view.addSubview(menuButton)
        
        // Анимируем саму кнопку
        snapBehavior = UISnapBehavior(item: menuButton, snapTo: CGPoint(x: view.center.x, y: 100))
        snapBehavior?.damping = 0.3 // Эффект пружины
    }
    
    @objc func toggleMenu() {
        // При каждом нажатии кнопка будет "прыгать" то в центр, то обратно
        guard let button = (snapBehavior?.items.first as? UIButton) else { return }
        let currentTarget = snapBehavior?.snapPoint ?? .zero
        let newTarget = currentTarget == CGPoint(x: view.center.x, y: 100) ? 
                        CGPoint(x: 20, y: 100) : 
                        CGPoint(x: view.center.x, y: 100)
        
        // Удаляем старый снэп и создаем новый
        if let oldBehavior = snapBehavior {
            animator.removeBehavior(oldBehavior)
        }
        snapBehavior = UISnapBehavior(item: button, snapTo: newTarget)
        snapBehavior?.damping = 0.5
        animator.addBehavior(snapBehavior!)
    }
}
```

---

### Продвинутый пример: Карточки с возвратом

Создадим карточку, которую можно перетаскивать, и которая плавно возвращается на место при отпускании.

```swift
import UIKit

class SnapCardViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var snapBehavior: UISnapBehavior?
    var cardView: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        
        // Создаем карточку
        cardView = UIView(frame: CGRect(x: 50, y: 200, width: 300, height: 200))
        cardView.backgroundColor = .systemOrange
        cardView.layer.cornerRadius = 20
        view.addSubview(cardView)
        
        // Инициализируем аниматор
        animator = UIDynamicAnimator(referenceView: view)
        
        // Добавляем гравитацию и столкновения для эффекта "падения" (опционально)
        let gravity = UIGravityBehavior(items: [cardView])
        gravity.gravityDirection = CGVector(dx: 0, dy: 0.5)
        animator.addBehavior(gravity)
        
        let collision = UICollisionBehavior(items: [cardView])
        collision.translatesReferenceBoundsIntoBoundary = true
        animator.addBehavior(collision)
        
        // Настраиваем жест перетаскивания
        let panGesture = UIPanGestureRecognizer(target: self, action: #selector(handlePan))
        cardView.addGestureRecognizer(panGesture)
    }
    
    @objc func handlePan(gesture: UIPanGestureRecognizer) {
        let translation = gesture.translation(in: view)
        
        switch gesture.state {
        case .changed:
            // Во время перетаскивания двигаем карточку вручную
            cardView.center = CGPoint(x: cardView.center.x + translation.x, 
                                      y: cardView.center.y + translation.y)
            gesture.setTranslation(.zero, in: view)
            
        case .ended:
            // При отпускании применяем эффект "прилипания" к исходной позиции
            let originalCenter = CGPoint(x: view.bounds.midX, y: 200 + cardView.bounds.height / 2)
            
            // Удаляем старый снэп, если был
            if let snap = snapBehavior {
                animator.removeBehavior(snap)
            }
            
            // Создаем новый снэп
            snapBehavior = UISnapBehavior(item: cardView, snapTo: originalCenter)
            snapBehavior?.damping = 0.4 // Эффект пружины
            animator.addBehavior(snapBehavior!)
        default: break
        }
    }
}
```

---

### Решение проблемы: Конфликт с другими поведениями

Если вы добавляете `UISnapBehavior` к объекту, на который уже действуют другие физические силы (например, гравитация или постоянный толчок), их эффекты могут накладываться и мешать.

**Решение:** Перед добавлением `UISnapBehavior` удалите конфликтующие поведения (или временно отключите их), а затем добавьте обратно после завершения анимации.

```swift
class CollisionHandlingViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var gravity: UIGravityBehavior!
    var snapBehavior: UISnapBehavior?
    
    func snapToCenter(viewToSnap: UIView) {
        // 1. Удаляем гравитацию, чтобы она не мешала
        if let g = gravity {
            animator.removeBehavior(g)
        }
        
        // 2. Применяем снэп
        let targetPoint = CGPoint(x: self.view.bounds.midX, y: 300)
        snapBehavior = UISnapBehavior(item: viewToSnap, snapTo: targetPoint)
        snapBehavior?.damping = 0.8
        
        // 3. По завершении возвращаем гравитацию
        snapBehavior?.action = { [weak self] in
            if let strongSelf = self, 
               let snap = strongSelf.snapBehavior,
               let view = snap.items.first as? UIView,
               view.center.distance(to: targetPoint) < 5 { // Остановился близко к цели
                
                strongSelf.animator.removeBehavior(snap)
                strongSelf.snapBehavior = nil
                if let g = strongSelf.gravity {
                    strongSelf.animator.addBehavior(g)
                }
            }
        }
        animator.addBehavior(snapBehavior!)
    }
}

extension CGPoint {
    func distance(to point: CGPoint) -> CGFloat {
        return sqrt(pow(x - point.x, 2) + pow(y - point.y, 2))
    }
}
```

---

### UISnapBehavior в UICollectionView

В кастомных `UICollectionViewFlowLayout` снэп-поведение часто используется для "притягивания" ячеек к определенной позиции при скролле (например, в каруселях).

```swift
class SnapFlowLayout: UICollectionViewFlowLayout {
    private var dynamicAnimator: UIDynamicAnimator!
    
    override func prepare() {
        super.prepare()
        guard let collectionView = collectionView, dynamicAnimator == nil else { return }
        
        dynamicAnimator = UIDynamicAnimator(collectionViewLayout: self)
        
        let visibleRect = CGRect(origin: collectionView.bounds.origin, 
                                 size: collectionView.frame.size)
        guard let attributesArray = super.layoutAttributesForElements(in: visibleRect) else { return }
        
        for attributes in attributesArray {
            let behavior = UISnapBehavior(item: attributes, snapTo: attributes.center)
            behavior.damping = 0.8
            dynamicAnimator.addBehavior(behavior)
        }
    }
}
```

---

### Лучшие практики

1.  **Удаляйте старые поведения:** Перед повторным применением `UISnapBehavior` к одному и тому же объекту всегда удаляйте предыдущий экземпляр из аниматора, иначе эффекты могут наложиться и вызвать дергание .
2.  **Не используйте для больших расстояний:** Снэп идеален для небольших смещений (меню, карточки). Для телепортации объектов на большое расстояние лучше использовать стандартную анимацию `UIView.animate`, так как физический просчет для длинных дистанций выглядит неестественно .
3.  **Комбинируйте с `UIDynamicItemBehavior`:** Для лучшего контроля добавьте [[UIDynamicItemBehavior]] к тому же элементу, чтобы настроить упругость или разрешить вращение во время "прыжка" .
4.  **Учитывайте `transform`:** Если вы применили к элементу трансформацию (скейл, поворот), обязательно вызовите `animator.updateItemUsingCurrentState(element)` перед добавлением снэп-поведения, чтобы аниматор учел изменения.

---

### Итог

**`UISnapBehavior`** — это простой, но мощный инструмент для создания "живых" интерфейсов. Он берет на себя сложные математические расчеты пружинного движения, позволяя разработчику сосредоточиться на логике приложения. Эффект "прилипания" отлично подходит для меню, карточек, диалоговых окон и интерактивных элементов, требующих плавной и физически правдоподобной обратной связи .