#core-animation #catransform3d #3d #animation #ios #swift #perspective

---

## CATransform3D — 3D-трансформации в Core Animation

### Определение

**`CATransform3D`** — это структура в [[Core Animation]], представляющая **трёхмерное аффинное преобразование** (3D affine transformation). Она позволяет выполнять в 3D-пространстве перемещение, масштабирование, поворот вокруг осей X, Y и Z, а также добавлять **перспективу** (perspective), создавая иллюзию глубины .

В отличие от [[CGAffineTransform]] (который работает только в 2D), `CATransform3D` даёт полный контроль над положением и ориентацией слоя в трёхмерном пространстве.

### Зачем это знать iOS-разработчику?

1.  **3D-анимации:** Вращение карточек, flip-анимации, эффект "куба".
2.  **Перспектива:** Создание иллюзии глубины (например, "бесконечная лента", "карусель").
3.  **3D-сцены:** Расположение слоёв в пространстве для создания простых 3D-интерфейсов.
4.  **Анимация переходов:** Кастомные переходы между экранами с 3D-эффектами.
5.  **Работа с `CATransformLayer`:** Для создания сложных 3D-иерархий.

---

### Математическая основа

`CATransform3D` описывается матрицей 4×4:

\[
\begin{bmatrix}
m11 & m12 & m13 & m14 \\
m21 & m22 & m23 & m24 \\
m31 & m32 & m33 & m34 \\
m41 & m42 & m43 & m44
\end{bmatrix}
\]

В Swift структура имеет 16 полей (`m11` ... `m44`), но для большинства задач используются удобные методы-конструкторы.

```swift
struct CATransform3D {
    var m11: CGFloat, m12: CGFloat, m13: CGFloat, m14: CGFloat
    var m21: CGFloat, m22: CGFloat, m23: CGFloat, m24: CGFloat
    var m31: CGFloat, m32: CGFloat, m33: CGFloat, m34: CGFloat
    var m41: CGFloat, m42: CGFloat, m43: CGFloat, m44: CGFloat
}
```

---

### Типы трансформаций

#### 1. **Перемещение (Translation)**

```swift
// Перемещение по X, Y, Z
let transform = CATransform3DMakeTranslation(x: 100, y: 50, z: 20)
// или модификация существующей
var transform = CATransform3DIdentity
transform = CATransform3DTranslate(transform, 100, 50, 20)
```

#### 2. **Масштабирование (Scale)**

```swift
let transform = CATransform3DMakeScale(x: 2.0, y: 1.5, z: 1.0)
```

#### 3. **Поворот (Rotation)**

```swift
// Поворот вокруг оси Z (как 2D-поворот)
let transformZ = CATransform3DMakeRotation(angle: .pi / 4, x: 0, y: 0, z: 1)

// Поворот вокруг оси X (наклон вперёд/назад)
let transformX = CATransform3DMakeRotation(angle: .pi / 4, x: 1, y: 0, z: 0)

// Поворот вокруг оси Y (вращение влево/вправо)
let transformY = CATransform3DMakeRotation(angle: .pi / 4, x: 0, y: 1, z: 0)
```

#### 4. **Перспектива (Perspective)**

Перспектива задаётся через элемент `m34`. Чем меньше значение, тем сильнее эффект глубины.

```swift
var perspective = CATransform3DIdentity
perspective.m34 = -1.0 / 500.0  // Типичное значение
```

---

### Композиция трансформаций

Порядок трансформаций важен. Используйте [[CATransform3DConcat]] для объединения.

```swift
var transform = CATransform3DIdentity
transform = CATransform3DTranslate(transform, 100, 0, 0)
transform = CATransform3DRotate(transform, .pi / 4, 0, 1, 0)
transform = CATransform3DScale(transform, 1.5, 1.5, 1.0)
```

---

### Базовые примеры

#### 1. **Поворот вокруг оси Y (3D-вращение)**

```swift
import UIKit

class Rotation3DViewController: UIViewController {
    var imageView: UIImageView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        imageView = UIImageView(frame: CGRect(x: 100, y: 200, width: 200, height: 200))
        imageView.image = UIImage(systemName: "star.fill")
        imageView.tintColor = .systemYellow
        imageView.contentMode = .scaleAspectFit
        view.addSubview(imageView)
        
        // Поворот на 180 градусов вокруг оси Y
        var transform = CATransform3DIdentity
        transform = CATransform3DRotate(transform, .pi, 0, 1, 0)
        imageView.layer.transform = transform
    }
}
```

#### 2. **3D-вращение с перспективой**

```swift
class PerspectiveRotationViewController: UIViewController {
    var imageView: UIImageView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        imageView = UIImageView(frame: CGRect(x: 100, y: 200, width: 200, height: 200))
        imageView.image = UIImage(systemName: "photo")
        imageView.tintColor = .systemBlue
        imageView.contentMode = .scaleAspectFit
        view.addSubview(imageView)
        
        // Добавляем перспективу
        var transform = CATransform3DIdentity
        transform.m34 = -1.0 / 500.0
        
        // Поворот на 45 градусов вокруг оси Y
        transform = CATransform3DRotate(transform, .pi / 4, 0, 1, 0)
        imageView.layer.transform = transform
    }
}
```

---

### Анимация 3D-трансформаций

```swift
class Transform3DAnimationViewController: UIViewController {
    var imageView: UIImageView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        imageView = UIImageView(frame: CGRect(x: 100, y: 200, width: 200, height: 200))
        imageView.image = UIImage(systemName: "heart.fill")
        imageView.tintColor = .systemRed
        imageView.contentMode = .scaleAspectFit
        view.addSubview(imageView)
        
        let button = UIButton(frame: CGRect(x: 50, y: 500, width: 200, height: 50))
        button.setTitle("Вращать", for: .normal)
        button.backgroundColor = .systemBlue
        button.addTarget(self, action: #selector(animate3DRotation), for: .touchUpInside)
        view.addSubview(button)
    }
    
    @objc func animate3DRotation() {
        let animation = CABasicAnimation(keyPath: "transform")
        
        var startTransform = CATransform3DIdentity
        startTransform.m34 = -1.0 / 500.0
        
        var endTransform = CATransform3DIdentity
        endTransform.m34 = -1.0 / 500.0
        endTransform = CATransform3DRotate(endTransform, .pi, 0, 1, 0)
        
        animation.fromValue = startTransform
        animation.toValue = endTransform
        animation.duration = 1.0
        animation.fillMode = .forwards
        animation.isRemovedOnCompletion = false
        
        imageView.layer.add(animation, forKey: "3dRotation")
    }
}
```

---

### Эффект "карточной колоды" (Card Deck)

```swift
class CardDeckViewController: UIViewController {
    var cards: [UIView] = []
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        for i in 0..<5 {
            let card = UIView(frame: CGRect(x: 50, y: 200 + CGFloat(i) * 10, width: 200, height: 150))
            card.backgroundColor = UIColor(red: CGFloat.random(in: 0...1),
                                          green: CGFloat.random(in: 0...1),
                                          blue: CGFloat.random(in: 0...1), alpha: 1)
            card.layer.cornerRadius = 12
            view.addSubview(card)
            cards.append(card)
            
            // Перспектива для каждой карты
            var transform = CATransform3DIdentity
            transform.m34 = -1.0 / 500.0
            transform = CATransform3DRotate(transform, CGFloat(i) * 0.1, 0, 1, 0)
            card.layer.transform = transform
        }
    }
}
```

---

### Flip-анимация (переворот карточки)

```swift
class FlipAnimationViewController: UIViewController {
    var frontView: UIView!
    var backView: UIView!
    var isFlipped = false
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Передняя сторона
        frontView = UIView(frame: CGRect(x: 100, y: 200, width: 200, height: 250))
        frontView.backgroundColor = .systemRed
        frontView.layer.cornerRadius = 12
        view.addSubview(frontView)
        
        // Задняя сторона (изначально скрыта)
        backView = UIView(frame: CGRect(x: 100, y: 200, width: 200, height: 250))
        backView.backgroundColor = .systemBlue
        backView.layer.cornerRadius = 12
        view.addSubview(backView)
        
        let button = UIButton(frame: CGRect(x: 50, y: 500, width: 200, height: 50))
        button.setTitle("Перевернуть", for: .normal)
        button.backgroundColor = .systemGreen
        button.addTarget(self, action: #selector(flipCard), for: .touchUpInside)
        view.addSubview(button)
    }
    
    @objc func flipCard() {
        let duration = 0.5
        
        if !isFlipped {
            UIView.transition(from: frontView, to: backView, duration: duration,
                            options: [.transitionFlipFromLeft, .showHideTransitionViews]) { _ in
                self.isFlipped = true
            }
        } else {
            UIView.transition(from: backView, to: frontView, duration: duration,
                            options: [.transitionFlipFromRight, .showHideTransitionViews]) { _ in
                self.isFlipped = false
            }
        }
    }
}
```

---

### CATransformLayer для 3D-групп

`CATransformLayer` позволяет создавать иерархии слоёв, где каждый слой имеет свою трансформацию, а также наследует трансформации родителя.

```swift
class Transform3DGroupViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let transformLayer = CATransformLayer()
        transformLayer.frame = view.bounds
        view.layer.addSublayer(transformLayer)
        
        let redLayer = CALayer()
        redLayer.bounds = CGRect(x: 0, y: 0, width: 100, height: 100)
        redLayer.position = CGPoint(x: 150, y: 200)
        redLayer.backgroundColor = UIColor.systemRed.cgColor
        transformLayer.addSublayer(redLayer)
        
        let blueLayer = CALayer()
        blueLayer.bounds = CGRect(x: 0, y: 0, width: 100, height: 100)
        blueLayer.position = CGPoint(x: 250, y: 200)
        blueLayer.backgroundColor = UIColor.systemBlue.cgColor
        transformLayer.addSublayer(blueLayer)
        
        // Вращаем всю группу
        var groupTransform = CATransform3DIdentity
        groupTransform.m34 = -1.0 / 500.0
        groupTransform = CATransform3DRotate(groupTransform, .pi / 4, 0, 1, 0)
        transformLayer.transform = groupTransform
    }
}
```

---

### Сброс трансформации

```swift
UIView.animate(withDuration: 0.3) {
    self.imageView.layer.transform = CATransform3DIdentity
}
```

---

### Работа с [[UIView]] (через layer)

`UIView` не имеет прямого свойства для 3D-трансформаций, но можно использовать `layer.transform`.

```swift
var transform = CATransform3DIdentity
transform.m34 = -1.0 / 500.0
transform = CATransform3DRotate(transform, .pi / 6, 1, 0, 0)
myView.layer.transform = transform
```

---

### Лучшие практики

1.  **Всегда устанавливайте перспективу (`m34`) для реалистичных 3D-эффектов.**
2.  **Используйте `CATransformLayer` для группировки 3D-объектов** — каждый слой будет правильно отображаться в пространстве.
3.  **Для flip-анимации используйте `UIView.transition`** — это проще и эффективнее ручной анимации.
4.  **Не злоупотребляйте 3D-трансформациями на старых устройствах** — это может снижать производительность.
5.  **Анимируйте `transform` через `CABasicAnimation`** для полного контроля.

---

### Итог

**`CATransform3D`** — это мощный инструмент для создания трёхмерных эффектов в iOS. Он позволяет:

1.  **Перемещать, масштабировать и вращать слои** в 3D-пространстве.
2.  **Добавлять перспективу** для реалистичной глубины.
3.  **Создавать сложные 3D-сцены** с помощью [[CATransformLayer]].
4.  **Анимировать 3D-трансформации** через [[CABasicAnimation]].
5.  **Реализовывать эффекты переворота, карусели, "куба"** и другие.

Понимание `CATransform3D` необходимо для создания впечатляющих 3D-анимаций и интерфейсов, выходящих за рамки плоскости экрана .