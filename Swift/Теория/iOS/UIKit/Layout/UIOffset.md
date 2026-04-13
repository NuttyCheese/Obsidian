#uikit #uioffset #uidynamics #layout #ios #swift

---

## UIOffset — Смещение в UIKit

### Определение

**`UIOffset`** — это структура в [[UIKit]], представляющая **смещение** (offset) по горизонтали (`horizontal`) и вертикали (`vertical`). В отличие от [[CGPoint]] (абсолютная позиция) или [[CGVector]] (направление и величина), `UIOffset` обычно используется для указания **относительного сдвига** относительно центра или другого опорного элемента.

Основное применение `UIOffset` — в **UIKit Dynamics** ([[UIAttachmentBehavior]], [[UIPushBehavior]]), а также в некоторых случаях в [[UICollectionViewFlowLayout]] и анимациях.

### Зачем это знать iOS-разработчику?

1.  **UIKit Dynamics:** Задание точки приложения силы относительно центра элемента ([[UIAttachmentBehavior]], [[UIPushBehavior]]).
2.  **Анимации:** Смещение элементов относительно их исходного положения.
3.  **UICollectionViewFlowLayout:** Свойство `sectionInset` использует `UIEdgeInsets`, но `UIOffset` иногда применяется для корректировки позиций.
4.  **Совместимость с Objective-C:** В некоторых старых API используется `UIOffset`.
5.  **Переиспользование кода:** Единообразный тип для смещений в UIKit.

---

### Создание UIOffset

```swift
import UIKit

// Стандартный инициализатор
let offset = UIOffset(horizontal: 10.0, vertical: 20.0)

// Нулевое смещение
let zeroOffset = UIOffset.zero

// Из CGSize (преобразование)
let size = CGSize(width: 15, height: 25)
let offsetFromSize = UIOffset(horizontal: size.width, vertical: size.height)

// Из CGPoint (преобразование)
let point = CGPoint(x: 5, y: 10)
let offsetFromPoint = UIOffset(horizontal: point.x, vertical: point.y)
```

---

### Свойства и методы

| Свойство / Метод | Тип         | Описание                         |
| ---------------- | ----------- | -------------------------------- |
| **`horizontal`** | [[CGFloat]] | Горизонтальное смещение.         |
| **`vertical`**   | `CGFloat`   | Вертикальное смещение.           |
| **`zero`**       | `UIOffset`  | Нулевое смещение (`(0.0, 0.0)`). |
| **`==`**         | [[Bool]]    | Сравнение двух `UIOffset`.       |

---

### Использование в UIKit Dynamics

#### 1. **UIAttachmentBehavior — смещение точки привязки**

```swift
import UIKit

class AttachmentOffsetViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var attachment: UIAttachmentBehavior?
    var square: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        square = UIView(frame: CGRect(x: 100, y: 200, width: 100, height: 100))
        square.backgroundColor = .systemBlue
        view.addSubview(square)
        
        animator = UIDynamicAnimator(referenceView: view)
        
        let pan = UIPanGestureRecognizer(target: self, action: #selector(handlePan))
        view.addGestureRecognizer(pan)
    }
    
    @objc func handlePan(_ gesture: UIPanGestureRecognizer) {
        let location = gesture.location(in: view)
        
        switch gesture.state {
        case .began:
            // Смещение от центра квадрата до точки касания
            let offset = UIOffset(
                horizontal: location.x - square.center.x,
                vertical: location.y - square.center.y
            )
            
            attachment = UIAttachmentBehavior(item: square, offsetFromCenter: offset, attachedToAnchor: location)
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

#### 2. **UIAttachmentBehavior с фиксированным смещением**

```swift
// Привязка к точке (смещение 20 пикселей вправо и 30 вниз от центра)
let offset = UIOffset(horizontal: 20, vertical: 30)
let attachment = UIAttachmentBehavior(item: view, offsetFromCenter: offset, attachedToAnchor: CGPoint(x: 100, y: 100))
```

#### 3. **Привязка двух элементов со смещениями**

```swift
let offset1 = UIOffset(horizontal: 10, vertical: 0)   // Смещение для первого элемента
let offset2 = UIOffset(horizontal: -10, vertical: 0)  // Смещение для второго элемента

let attachment = UIAttachmentBehavior(item: view1, offsetFromCenter: offset1,
                                      attachedTo: view2, offsetFromCenter: offset2)
```

#### 4. **UIPushBehavior — смещение точки приложения силы**

```swift
let push = UIPushBehavior(items: [ball], mode: .instantaneous)
push.targetOffset = UIOffset(horizontal: 20, vertical: 0)  // Сила приложена с отступом 20 от центра
push.angle = 0  // Горизонтальный толчок
push.magnitude = 2.0
animator.addBehavior(push)
```

---

### UIOffset в UICollectionViewFlowLayout

Хотя `UIOffset` редко используется напрямую в `UICollectionView`, его можно применять для корректировки позиций.

```swift
class OffsetCollectionLayout: UICollectionViewFlowLayout {
    var offset: UIOffset = .zero
    
    override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
        guard let attributes = super.layoutAttributesForElements(in: rect) else { return nil }
        
        for attribute in attributes {
            attribute.frame = attribute.frame.offsetBy(dx: offset.horizontal, dy: offset.vertical)
        }
        
        return attributes
    }
}
```

---

### UIOffset в анимациях

```swift
class OffsetAnimationViewController: UIViewController {
    var animatedView: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        animatedView = UIView(frame: CGRect(x: 100, y: 200, width: 100, height: 100))
        animatedView.backgroundColor = .systemRed
        view.addSubview(animatedView)
        
        let button = UIButton(frame: CGRect(x: 50, y: 500, width: 200, height: 50))
        button.setTitle("Сдвинуть", for: .normal)
        button.backgroundColor = .systemBlue
        button.addTarget(self, action: #selector(animateOffset), for: .touchUpInside)
        view.addSubview(button)
    }
    
    @objc func animateOffset() {
        let offset = UIOffset(horizontal: 50, vertical: 30)
        
        UIView.animate(withDuration: 0.5) {
            self.animatedView.transform = CGAffineTransform(translationX: offset.horizontal, y: offset.vertical)
        } completion: { _ in
            UIView.animate(withDuration: 0.5) {
                self.animatedView.transform = .identity
            }
        }
    }
}
```

---

### Преобразование между UIOffset и [[CGPoint]]/[[CGSize]]

```swift
extension UIOffset {
    func toPoint() -> CGPoint {
        return CGPoint(x: horizontal, y: vertical)
    }
    
    func toSize() -> CGSize {
        return CGSize(width: horizontal, height: vertical)
    }
}

extension CGPoint {
    func toOffset() -> UIOffset {
        return UIOffset(horizontal: x, vertical: y)
    }
}

extension CGSize {
    func toOffset() -> UIOffset {
        return UIOffset(horizontal: width, vertical: height)
    }
}

// Использование
let offset = UIOffset(horizontal: 10, vertical: 20)
let point = offset.toPoint()      // (10, 20)
let size = offset.toSize()        // (10, 20)
```

---

### UIOffset vs CGPoint vs CGVector

| Тип            | Значение                     | Применение                                                |
| -------------- | ---------------------------- | --------------------------------------------------------- |
| **`UIOffset`** | Смещение относительно центра | UIKit Dynamics (`UIAttachmentBehavior`, `UIPushBehavior`) |
| **`CGPoint`**  | Абсолютная позиция           | Расположение элементов, `frame.origin`, `center`          |
| **`CGVector`** | Направление и сила           | Гравитация, толчки, скорости                              |

---

### UIOffsetZero и пустое смещение

```swift
let zero = UIOffset.zero  // (0.0, 0.0)

let isZero = offset == .zero  // true, если offset == (0,0)
```

---

### Работа с UIOffset в кастомных классах

```swift
class CustomView: UIView {
    var offset: UIOffset = .zero {
        didSet {
            updatePosition()
        }
    }
    
    private func updatePosition() {
        transform = CGAffineTransform(translationX: offset.horizontal, y: offset.vertical)
    }
    
    func resetOffset() {
        offset = .zero
    }
}
```

---

### Лучшие практики

1.  **Используйте `UIOffset.zero` вместо `UIOffset(horizontal: 0, vertical: 0)`.**
2.  **При работе с UIKit Dynamics всегда задавайте `targetOffset` для точного контроля.**
3.  **Для абсолютных позиций используйте `CGPoint`, для направлений — `CGVector`.**
4.  **При анимациях учитывайте, что `UIOffset` — это относительное смещение.**

```swift
// Правильно: используем .zero
attachment = UIAttachmentBehavior(item: view, offsetFromCenter: .zero, attachedToAnchor: point)

// Неправильно: создаём вручную
attachment = UIAttachmentBehavior(item: view, offsetFromCenter: UIOffset(horizontal: 0, vertical: 0), attachedToAnchor: point)
```

---

### Итог

**`UIOffset`** — это специализированный тип для представления относительного смещения в UIKit. Он используется в основном в:

1.  **UIKit Dynamics:** Точка приложения силы в `UIAttachmentBehavior` и `UIPushBehavior`.
2.  **Кастомных анимациях:** Относительное смещение элементов.
3.  **Корректировке позиций** в `UICollectionViewFlowLayout`.

Понимание `UIOffset` необходимо для тонкой настройки физических поведений в UIKit Dynamics и создания сложных анимаций с относительными смещениями.