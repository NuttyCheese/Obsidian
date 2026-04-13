#core-animation #catransform3d #3d #animation #ios #swift #math #transformation

---

## CATransform3DConcat — Объединение 3D-трансформаций

### Определение

**`CATransform3DConcat`** — это функция в Core Animation, которая объединяет (конкатенирует) две 3D-трансформации в одну результирующую. Математически это **матричное умножение** двух `CATransform3D`: `CATransform3DConcat(t2, t1)` = `t2 * t1` (применяется `t1`, затем `t2`) .

Это основной способ комбинирования нескольких трансформаций (перемещение, поворот, масштабирование, перспектива) в одну, которая затем применяется к слою.

### Зачем это знать iOS-разработчику?

1.  **Комбинация трансформаций:** Объединение перемещения, поворота и масштабирования в одну трансформацию.
2.  **Порядок имеет значение:** `CATransform3DConcat` даёт точный контроль над порядком применения.
3.  **Иерархии слоёв:** Применение трансформаций к родительским и дочерним слоям.
4.  **Производительность:** Одна объединённая трансформация эффективнее, чем несколько отдельных.
5.  **Анимации:** Интерполяция между двумя сложными трансформациями.

---

### Синтаксис и математика

```swift
func CATransform3DConcat(_ a: CATransform3D, _ b: CATransform3D) -> CATransform3D
```

**Важно:** Результат = `a * b`, то есть **сначала применяется `b`, затем `a`**.

```swift
let transform = CATransform3DConcat(rotate, move)
// Эквивалентно: move, потом rotate
```

---

### CATransform3DConcat vs Цепочка методов

| Способ | Код | Порядок |
|---|---|---|
| **Цепочка** | `var t = CATransform3DIdentity; t = CATransform3DTranslate(t, x, y, z); t = CATransform3DRotate(t, angle, 0, 1, 0)` | Сначала `translate`, потом `rotate` |
| **Concat** | `CATransform3DConcat(rotate, translate)` | Сначала `translate`, потом `rotate` (т.к. `rotate * translate`) |

Оба способа эквивалентны, но `Concat` удобен для динамического объединения трансформаций, полученных из разных источников.

---

### Базовые примеры

#### 1. **Перемещение + Поворот**

```swift
import UIKit

class TransformConcatViewController: UIViewController {
    var imageView: UIImageView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        imageView = UIImageView(frame: CGRect(x: 100, y: 200, width: 150, height: 150))
        imageView.image = UIImage(systemName: "arrow.right.circle.fill")
        imageView.tintColor = .systemBlue
        imageView.contentMode = .scaleAspectFit
        view.addSubview(imageView)
        
        // Трансформации
        let translate = CATransform3DMakeTranslation(50, 100, 0)
        let rotate = CATransform3DMakeRotation(.pi / 4, 0, 1, 0)
        
        // Сначала перемещение, потом поворот
        let transform = CATransform3DConcat(rotate, translate)
        imageView.layer.transform = transform
    }
}
```

#### 2. **Поворот + Масштабирование + Перспектива**

```swift
class TransformConcatAdvancedViewController: UIViewController {
    var imageView: UIImageView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        imageView = UIImageView(frame: CGRect(x: 100, y: 200, width: 150, height: 150))
        imageView.image = UIImage(systemName: "star.fill")
        imageView.tintColor = .systemYellow
        imageView.contentMode = .scaleAspectFit
        view.addSubview(imageView)
        
        // Перспектива
        var perspective = CATransform3DIdentity
        perspective.m34 = -1.0 / 500.0
        
        // Поворот вокруг Y
        let rotate = CATransform3DMakeRotation(.pi / 3, 0, 1, 0)
        
        // Масштабирование
        let scale = CATransform3DMakeScale(1.5, 1.5, 1.0)
        
        // Объединяем: сначала перспектива, потом поворот, потом масштабирование
        var transform = CATransform3DConcat(perspective, rotate)
        transform = CATransform3DConcat(scale, transform)
        
        imageView.layer.transform = transform
    }
}
```

---

### Порядок трансформаций (эксперимент)

```swift
class TransformOrderViewController: UIViewController {
    @IBOutlet weak var box1: UIView!
    @IBOutlet weak var box2: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // box1: сначала перемещение, потом поворот
        let translate = CATransform3DMakeTranslation(100, 0, 0)
        let rotate = CATransform3DMakeRotation(.pi / 2, 0, 0, 1)
        box1.layer.transform = CATransform3DConcat(rotate, translate)
        
        // box2: сначала поворот, потом перемещение (другой результат!)
        box2.layer.transform = CATransform3DConcat(translate, rotate)
    }
}
```

---

### Анимация с CATransform3DConcat

```swift
class AnimatedTransformConcatViewController: UIViewController {
    var imageView: UIImageView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        imageView = UIImageView(frame: CGRect(x: 100, y: 200, width: 150, height: 150))
        imageView.image = UIImage(systemName: "heart.fill")
        imageView.tintColor = .systemRed
        imageView.contentMode = .scaleAspectFit
        view.addSubview(imageView)
        
        let button = UIButton(frame: CGRect(x: 50, y: 500, width: 200, height: 50))
        button.setTitle("Анимировать", for: .normal)
        button.backgroundColor = .systemBlue
        button.addTarget(self, action: #selector(animateTransform), for: .touchUpInside)
        view.addSubview(button)
    }
    
    @objc func animateTransform() {
        let startTransform = imageView.layer.transform
        
        var perspective = CATransform3DIdentity
        perspective.m34 = -1.0 / 500.0
        
        let rotate = CATransform3DMakeRotation(.pi, 0, 1, 0)
        let scale = CATransform3DMakeScale(0.5, 0.5, 1.0)
        
        let midTransform = CATransform3DConcat(rotate, perspective)
        let endTransform = CATransform3DConcat(scale, midTransform)
        
        let animation = CABasicAnimation(keyPath: "transform")
        animation.fromValue = startTransform
        animation.toValue = endTransform
        animation.duration = 1.0
        animation.fillMode = .forwards
        animation.isRemovedOnCompletion = false
        
        imageView.layer.add(animation, forKey: "transformAnimation")
    }
}
```

---

### Использование в иерархиях слоёв

```swift
class HierarchicalTransformViewController: UIViewController {
    var parentLayer: CALayer!
    var childLayer: CALayer!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        parentLayer = CALayer()
        parentLayer.frame = CGRect(x: 100, y: 200, width: 200, height: 200)
        parentLayer.backgroundColor = UIColor.systemGray.cgColor
        view.layer.addSublayer(parentLayer)
        
        childLayer = CALayer()
        childLayer.frame = CGRect(x: 50, y: 50, width: 100, height: 100)
        childLayer.backgroundColor = UIColor.systemBlue.cgColor
        parentLayer.addSublayer(childLayer)
        
        // Трансформация родителя
        let parentRotate = CATransform3DMakeRotation(.pi / 6, 0, 0, 1)
        parentLayer.transform = parentRotate
        
        // Трансформация ребёнка (относительно родителя)
        let childTranslate = CATransform3DMakeTranslation(30, 30, 0)
        childLayer.transform = childTranslate
    }
}
```

---

### Комбинирование с анимацией ключевых кадров

```swift
class KeyframeTransformViewController: UIViewController {
    var imageView: UIImageView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        imageView = UIImageView(frame: CGRect(x: 100, y: 200, width: 150, height: 150))
        imageView.image = UIImage(systemName: "camera.fill")
        imageView.tintColor = .systemGreen
        imageView.contentMode = .scaleAspectFit
        view.addSubview(imageView)
        
        let animation = CAKeyframeAnimation(keyPath: "transform")
        
        let identity = CATransform3DIdentity
        
        var perspective = CATransform3DIdentity
        perspective.m34 = -1.0 / 500.0
        
        let rotateY = CATransform3DMakeRotation(.pi / 2, 0, 1, 0)
        let rotateX = CATransform3DMakeRotation(.pi / 2, 1, 0, 0)
        let scale = CATransform3DMakeScale(0.5, 0.5, 1.0)
        
        animation.values = [
            identity,
            CATransform3DConcat(rotateY, perspective),
            CATransform3DConcat(rotateX, perspective),
            CATransform3DConcat(scale, CATransform3DConcat(rotateY, perspective)),
            identity
        ]
        
        animation.keyTimes = [0, 0.25, 0.5, 0.75, 1]
        animation.duration = 2.0
        animation.repeatCount = .infinity
        
        imageView.layer.add(animation, forKey: "keyframe3d")
    }
}
```

---

### CATransform3DConcat с CATransformLayer

```swift
class TransformLayerGroupViewController: UIViewController {
    var transformLayer: CATransformLayer!
    var layers: [CALayer] = []
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        transformLayer = CATransformLayer()
        transformLayer.frame = view.bounds
        view.layer.addSublayer(transformLayer)
        
        for i in 0..<5 {
            let layer = CALayer()
            layer.bounds = CGRect(x: 0, y: 0, width: 60, height: 60)
            layer.position = CGPoint(x: 100 + CGFloat(i) * 80, y: 300)
            layer.backgroundColor = UIColor(hue: CGFloat(i) / 5, saturation: 0.8, brightness: 0.9, alpha: 1).cgColor
            layer.cornerRadius = 30
            transformLayer.addSublayer(layer)
            layers.append(layer)
        }
        
        // Групповая трансформация
        var groupTransform = CATransform3DIdentity
        groupTransform.m34 = -1.0 / 500.0
        groupTransform = CATransform3DRotate(groupTransform, .pi / 6, 0, 1, 0)
        
        let translate = CATransform3DMakeTranslation(0, -100, 0)
        let finalTransform = CATransform3DConcat(translate, groupTransform)
        
        transformLayer.transform = finalTransform
    }
}
```

---

### Лучшие практики

1.  **Для простых цепочек используйте цепочку методов:** `CATransform3DRotate(CATransform3DTranslate(...))` более читаема.
2.  **Для динамических трансформаций (полученных из разных источников) используйте `CATransform3DConcat`.**
3.  **Порядок конкатенации:** `CATransform3DConcat(t2, t1)` = сначала `t1`, потом `t2`.
4.  **Всегда добавляйте перспективу (`m34`) первой** — она должна быть базовой трансформацией.
5.  **Для анимаций между двумя сложными трансформациями используйте `CABasicAnimation` с `fromValue` и `toValue`.**

---

### Распространённые ошибки

1.  **Неправильный порядок конкатенации** — приводит к неожиданному поведению.
2.  **Забытая перспектива** — повороты выглядят как сплющенные 2D-трансформации.
3.  **Многократная конкатенация в цикле** — может накапливать ошибки округления.

---

### Итог

**`CATransform3DConcat`** — это мощный инструмент для объединения 3D-трансформаций в Core Animation. Он позволяет:

1.  **Комбинировать** перемещение, поворот, масштабирование и перспективу.
2.  **Контролировать порядок** применения трансформаций.
3.  **Анимировать** между сложными состояниями.
4.  **Эффективно работать** с иерархиями слоёв.
5.  **Динамически собирать трансформации** из разных частей кода.

Понимание `CATransform3DConcat` необходимо для создания сложных 3D-анимаций и управления трансформациями в Core Animation .