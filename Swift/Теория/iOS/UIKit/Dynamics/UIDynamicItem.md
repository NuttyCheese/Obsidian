#uikit #uidynamics #physics #protocol #animation #ios #swift

---

## UIDynamicItem — Протокол для физической анимации

### Определение

**`UIDynamicItem`** — это протокол в [[UIKit]], который определяет минимальные требования для объекта, чтобы он мог участвовать в физической анимации [[UIDynamicAnimator]]. Любой объект, соответствующий этому протоколу, может быть добавлен в такие поведения, как [[UIGravityBehavior]], [[UICollisionBehavior]], [[UIAttachmentBehavior]] и другие .

По умолчанию [[UIView]] и [[UICollectionViewLayoutAttributes]] уже соответствуют этому протоколу, но вы можете реализовать его для своих собственных классов, чтобы анимировать, например, [[CALayer]] или кастомные графические объекты.

### Зачем это знать iOS-разработчику?

1.  **Анимация не-UIView объектов:** Можно анимировать `CALayer`, [[CAShapeLayer]], кастомные модели.
2.  **Кастомные элементы управления:** Создание физических объектов с нестандартной геометрией.
3.  **Производительность:** Для сложной графики может быть выгоднее анимировать `CALayer` напрямую, минуя `UIView`.
4.  **Игровые движки:** При создании игр на UIKit может потребоваться кастомная физика.
5.  **Расширяемость:** Понимание протокола позволяет адаптировать любые объекты под систему UIKit Dynamics.

---

### Требования протокола

```swift
protocol UIDynamicItem : NSObjectProtocol {
    var bounds: CGRect { get }
    var center: CGPoint { get set }
    var transform: CGAffineTransform { get set }
}
```

| Свойство        | Тип                   | Описание                                                                                                                         |
| --------------- | --------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| **`bounds`**    | [[CGRect]]            | Прямоугольник, определяющий размер и форму объекта в его локальной системе координат (только чтение). Используется для коллизий. |
| **`center`**    | [[CGPoint]]           | Центр объекта в системе координат reference view (чтение и запись). Аниматор обновляет это свойство в каждом кадре.              |
| **`transform`** | [[CGAffineTransform]] | Трансформация объекта (поворот, масштаб). По умолчанию `.identity`.                                                              |

---

### UIView уже соответствует UIDynamicItem

Поскольку `UIView` уже реализует этот протокол, вы можете сразу использовать его в физической анимации:

```swift
let ball = UIView(frame: CGRect(x: 100, y: 100, width: 50, height: 50))
ball.backgroundColor = .systemBlue
ball.layer.cornerRadius = 25

let animator = UIDynamicAnimator(referenceView: view)
let gravity = UIGravityBehavior(items: [ball])
animator.addBehavior(gravity)  // ball соответствует UIDynamicItem
```

---

### Кастомная реализация: Анимация CALayer

`CALayer` не соответствует [[UIDynamicItem]], но мы можем создать обёртку.

```swift
class DynamicLayerItem: NSObject, UIDynamicItem {
    let layer: CALayer
    
    init(layer: CALayer) {
        self.layer = layer
        super.init()
    }
    
    var bounds: CGRect {
        return layer.bounds
    }
    
    var center: CGPoint {
        get {
            return CGPoint(x: layer.position.x, y: layer.position.y)
        }
        set {
            layer.position = CGPoint(x: newValue.x, y: newValue.y)
        }
    }
    
    var transform: CGAffineTransform {
        get {
            return layer.affineTransform()
        }
        set {
            layer.setAffineTransform(newValue)
        }
    }
}
```

#### Использование:

```swift
import UIKit

class LayerAnimationViewController: UIViewController {
    var animator: UIDynamicAnimator!
    var dynamicItem: DynamicLayerItem!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Создаем CALayer
        let shapeLayer = CAShapeLayer()
        shapeLayer.path = UIBezierPath(ovalIn: CGRect(x: 0, y: 0, width: 60, height: 60)).cgPath
        shapeLayer.fillColor = UIColor.systemRed.cgColor
        shapeLayer.position = CGPoint(x: 100, y: 100)
        view.layer.addSublayer(shapeLayer)
        
        // Оборачиваем в UIDynamicItem
        dynamicItem = DynamicLayerItem(layer: shapeLayer)
        
        animator = UIDynamicAnimator(referenceView: view)
        
        let gravity = UIGravityBehavior(items: [dynamicItem])
        animator.addBehavior(gravity)
        
        let collision = UICollisionBehavior(items: [dynamicItem])
        collision.translatesReferenceBoundsIntoBoundary = true
        animator.addBehavior(collision)
    }
}
```

---

### Кастомная реализация: Анимация CAShapeLayer с пользовательской геометрией

```swift
class ShapeDynamicItem: NSObject, UIDynamicItem {
    let shapeLayer: CAShapeLayer
    
    init(shapeLayer: CAShapeLayer) {
        self.shapeLayer = shapeLayer
        super.init()
    }
    
    var bounds: CGRect {
        return shapeLayer.bounds
    }
    
    var center: CGPoint {
        get {
            return CGPoint(x: shapeLayer.position.x, y: shapeLayer.position.y)
        }
        set {
            shapeLayer.position = CGPoint(x: newValue.x, y: newValue.y)
        }
    }
    
    var transform: CGAffineTransform {
        get {
            return shapeLayer.affineTransform()
        }
        set {
            shapeLayer.setAffineTransform(newValue)
        }
    }
}

// Использование
let starPath = UIBezierPath(starIn: CGRect(x: 0, y: 0, width: 50, height: 50))
let starLayer = CAShapeLayer()
starLayer.path = starPath.cgPath
starLayer.fillColor = UIColor.systemYellow.cgColor
starLayer.position = CGPoint(x: 150, y: 150)
view.layer.addSublayer(starLayer)

let starItem = ShapeDynamicItem(shapeLayer: starLayer)
animator.addBehavior(UIGravityBehavior(items: [starItem]))
```

---

### Анимация UICollectionViewLayoutAttributes

В кастомных [[UICollectionViewFlowLayout]] атрибуты ячеек также соответствуют `UIDynamicItem`.

```swift
class DynamicCollectionViewLayout: UICollectionViewFlowLayout {
    private var dynamicAnimator: UIDynamicAnimator!
    
    override func prepare() {
        super.prepare()
        guard let collectionView = collectionView, dynamicAnimator == nil else { return }
        
        dynamicAnimator = UIDynamicAnimator(collectionViewLayout: self)
        
        let visibleRect = CGRect(origin: collectionView.bounds.origin,
                                 size: collectionView.frame.size)
        guard let attributesArray = super.layoutAttributesForElements(in: visibleRect) else { return }
        
        for attributes in attributesArray {
            let behavior = UIAttachmentBehavior(item: attributes, attachedToAnchor: attributes.center)
            behavior.length = 0
            behavior.damping = 0.8
            behavior.frequency = 1.0
            dynamicAnimator.addBehavior(behavior)
        }
    }
}
```

---

### Расширения для UIDynamicItem

```swift
extension UIDynamicItem {
    /// Угол поворота объекта в радианах
    var rotationAngle: CGFloat {
        return atan2(transform.b, transform.a)
    }
    
    /// Скорость объекта (требует UIDynamicItemBehavior)
    func linearVelocity(for behavior: UIDynamicItemBehavior) -> CGPoint {
        return behavior.linearVelocity(for: self)
    }
}
```

---

### Лучшие практики

1.  **Анимируйте центр, а не frame:** Аниматор обновляет `center`, поэтому ваш объект должен правильно реагировать на изменение этого свойства.
2.  **Реализуйте bounds для коллизий:** `UICollisionBehavior` использует `bounds` для определения формы объекта. Если вы анимируете `CALayer`, убедитесь, что `bounds` корректно отражает геометрию.
3.  **Учитывайте transform:** Если ваша анимация требует вращения, аниматор будет обновлять `transform`. Убедитесь, что ваша реализация правильно применяет трансформации.
4.  **Не создавайте слишком много кастомных объектов:** Каждый `UIDynamicItem` добавляет нагрузку на физический движок. Для большого количества частиц рассмотрите [[CAReplicatorLayer]] или [[Metal]].

---

### Итог

**`UIDynamicItem`** — это фундаментальный протокол, который позволяет интегрировать любые объекты в систему физической анимации UIKit Dynamics. Понимание этого протокола открывает возможности:

1.  **Анимировать CALayer и другие объекты** без создания UIView.
2.  **Создавать кастомные физические элементы** с нестандартной геометрией.
3.  **Использовать UIKit Dynamics в играх** и сложных анимациях.
4.  **Расширять систему** под свои нужды.

Хотя в большинстве случаев `UIView` достаточно, `UIDynamicItem` даёт полный контроль над тем, какие объекты участвуют в физической симуляции .