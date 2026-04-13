#core-graphics #cgaffinetransform #animation #uikit #ios #swift #math

---

## CGAffineTransform — Аффинные преобразования в iOS

### Определение

**`CGAffineTransform`** — это структура в [[Core Graphics]], представляющая **аффинное преобразование** (affine transformation) — математическую операцию, которая отображает точки одной координатной системы в другую, сохраняя прямые линии и параллельность . Она позволяет выполнять линейные преобразования: перемещение, масштабирование, поворот и сдвиг (skew).

`CGAffineTransform` используется повсеместно в [[iOS]]: для анимации [[UIView]], трансформации [[CALayer]], работы с [[CGContext]] (Core Graphics) и обработки жестов.

### Зачем это знать iOS-разработчику?

1.  **Анимация трансформаций:** Плавное перемещение, увеличение, вращение элементов интерфейса.
2.  **Обработка жестов:** Применение трансформаций к вью в ответ на [[UIPanGestureRecognizer]], [[UIPinchGestureRecognizer]], [[UIRotationGestureRecognizer]].
3.  **Системы координат:** Преобразование координат между разными вью.
4.  **Core Graphics:** Рисование трансформированных фигур в `draw(_:)`.
5.  **Игровая механика:** Вращение спрайтов, масштабирование объектов.

---

### Математическая основа

Аффинное преобразование в 2D описывается матрицей 3×3 (последняя строка всегда `[0, 0, 1]`):
\[
\begin{bmatrix}
a & b & 0 \\
c & d & 0 \\
t_x & t_y & 1
\end{bmatrix}
\]

- **`a`, `b`, `c`, `d`** — отвечают за масштабирование, поворот и сдвиг.
- **`t_x`, `t_y`** — отвечают за перемещение (трансляцию).

`CGAffineTransform` в Swift хранит эти шесть значений:

```swift
struct CGAffineTransform {
    var a: CGFloat
    var b: CGFloat
    var c: CGFloat
    var d: CGFloat
    var tx: CGFloat
    var ty: CGFloat
}
```

---

### Типы трансформаций

#### 1. **Перемещение (Translation)**

```swift
let transform = CGAffineTransform(translationX: 100, y: 50)
```

Матрица:
\[
\begin{bmatrix}
1 & 0 & 0 \\
0 & 1 & 0 \\
100 & 50 & 1
\end{bmatrix}
\]

#### 2. **Масштабирование (Scale)**

```swift
let transform = CGAffineTransform(scaleX: 2.0, y: 1.5)
```

Матрица:
\[
\begin{bmatrix}
2 & 0 & 0 \\
0 & 1.5 & 0 \\
0 & 0 & 1
\end{bmatrix}
\]

#### 3. **Поворот (Rotation)**

```swift
let angle = CGFloat.pi / 4  // 45 градусов
let transform = CGAffineTransform(rotationAngle: angle)
```

Матрица:
\[
\begin{bmatrix}
\cos\theta & \sin\theta & 0 \\
-\sin\theta & \cos\theta & 0 \\
0 & 0 & 1
\end{bmatrix}
\]

#### 4. **Сдвиг (Skew)**

Сдвиг не имеет отдельного конструктора, но создаётся через `a`, `b`, `c`, `d`.

```swift
// Сдвиг по X
let skewX = CGAffineTransform(a: 1, b: 0, c: 0.5, d: 1, tx: 0, ty: 0)

// Сдвиг по Y
let skewY = CGAffineTransform(a: 1, b: 0.5, c: 0, d: 1, tx: 0, ty: 0)
```

---

### Композиция трансформаций (Concatenation)

Порядок трансформаций **важен**: `transform.concatenating(other)` применяет `other` **после** `transform` (справа налево в матричном умножении).

```swift
// Сначала масштабируем, потом поворачиваем, потом перемещаем
var transform = CGAffineTransform.identity
transform = transform.scaledBy(x: 2.0, y: 2.0)
transform = transform.rotated(by: .pi / 4)
transform = transform.translatedBy(x: 100, y: 50)
```

#### Удобные методы для композиции (возвращают новую трансформацию)

| Метод | Описание |
|---|---|
| **`translatedBy(x:y:)`** | Добавляет перемещение |
| **`scaledBy(x:y:)`** | Добавляет масштабирование |
| **`rotated(by:)`** | Добавляет поворот |
| **`concatenating(_:)`** | Объединяет с другой трансформацией |
| **`inverted()`** | Возвращает обратную трансформацию |

---

### Базовые примеры

#### 1. **Перемещение UIView**

```swift
import UIKit

class TransformViewController: UIViewController {
    var square: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        square = UIView(frame: CGRect(x: 100, y: 100, width: 100, height: 100))
        square.backgroundColor = .systemBlue
        view.addSubview(square)
        
        // Перемещение
        square.transform = CGAffineTransform(translationX: 50, y: 100)
    }
}
```

#### 2. **Масштабирование**

```swift
// Увеличение в 2 раза
square.transform = CGAffineTransform(scaleX: 2.0, y: 2.0)

// Растяжение по горизонтали
square.transform = CGAffineTransform(scaleX: 1.5, y: 1.0)
```

#### 3. **Поворот**

```swift
// Поворот на 45 градусов
square.transform = CGAffineTransform(rotationAngle: .pi / 4)
```

---

### Анимация трансформаций

```swift
class TransformAnimationViewController: UIViewController {
    var square: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        square = UIView(frame: CGRect(x: 100, y: 200, width: 100, height: 100))
        square.backgroundColor = .systemGreen
        view.addSubview(square)
        
        let button = UIButton(frame: CGRect(x: 50, y: 500, width: 200, height: 50))
        button.setTitle("Анимировать", for: .normal)
        button.backgroundColor = .systemBlue
        button.addTarget(self, action: #selector(animate), for: .touchUpInside)
        view.addSubview(button)
    }
    
    @objc func animate() {
        UIView.animate(withDuration: 0.5) {
            // Комбинация трансформаций
            var transform = CGAffineTransform.identity
            transform = transform.scaledBy(x: 1.5, y: 1.5)
            transform = transform.rotated(by: .pi)
            transform = transform.translatedBy(x: 50, y: 50)
            self.square.transform = transform
        } completion: { _ in
            UIView.animate(withDuration: 0.5) {
                self.square.transform = .identity
            }
        }
    }
}
```

---

### Работа с жестами

#### 1. **Pinch (масштабирование)**

```swift
@objc func handlePinch(_ gesture: UIPinchGestureRecognizer) {
    let scale = gesture.scale
    imageView.transform = CGAffineTransform(scaleX: scale, y: scale)
}
```

#### 2. **Rotation (поворот)**

```swift
@objc func handleRotation(_ gesture: UIRotationGestureRecognizer) {
    let rotation = gesture.rotation
    imageView.transform = CGAffineTransform(rotationAngle: rotation)
}
```

#### 3. **Pan (перемещение)**

```swift
@objc func handlePan(_ gesture: UIPanGestureRecognizer) {
    let translation = gesture.translation(in: view)
    imageView.transform = CGAffineTransform(translationX: translation.x, y: translation.y)
}
```

#### 4. **Комбинированный жест (одновременные трансформации)**

```swift
class CombinedGesturesViewController: UIViewController {
    @IBOutlet weak var imageView: UIImageView!
    var currentScale: CGFloat = 1.0
    var currentRotation: CGFloat = 0.0
    var currentTranslation: CGPoint = .zero
    
    @objc func handlePinch(_ gesture: UIPinchGestureRecognizer) {
        currentScale *= gesture.scale
        gesture.scale = 1.0
        applyTransform()
    }
    
    @objc func handleRotation(_ gesture: UIRotationGestureRecognizer) {
        currentRotation += gesture.rotation
        gesture.rotation = 0
        applyTransform()
    }
    
    @objc func handlePan(_ gesture: UIPanGestureRecognizer) {
        let translation = gesture.translation(in: view)
        currentTranslation.x += translation.x
        currentTranslation.y += translation.y
        gesture.setTranslation(.zero, in: view)
        applyTransform()
    }
    
    func applyTransform() {
        var transform = CGAffineTransform.identity
        transform = transform.scaledBy(x: currentScale, y: currentScale)
        transform = transform.rotated(by: currentRotation)
        transform = transform.translatedBy(x: currentTranslation.x, y: currentTranslation.y)
        imageView.transform = transform
    }
}
```

---

### Обратная трансформация (Inverse)

```swift
let originalTransform = CGAffineTransform(scaleX: 2, y: 2)
let inverted = originalTransform.inverted()

// Применение: точка в трансформированной системе -> точка в исходной
let transformedPoint = CGPoint(x: 100, y: 100)
let originalPoint = transformedPoint.applying(inverted)
print(originalPoint)  // (50, 50)
```

---

### Трансформация и точка привязки (anchorPoint)

По умолчанию трансформации применяются относительно центра `bounds` (`anchorPoint = (0.5, 0.5)`). Изменение `anchorPoint` меняет точку вращения/масштабирования.

```swift
// Вращение вокруг левого верхнего угла
imageView.layer.anchorPoint = CGPoint(x: 0, y: 0)
imageView.transform = CGAffineTransform(rotationAngle: .pi / 4)
```

---

### CGAffineTransform и CALayer

`CALayer` также поддерживает трансформации, но в 3D (`CATransform3D`). 2D-трансформации можно задать через `sublayerTransform` или преобразование [[CATransform3D]] из `CGAffineTransform`.

```swift
let affineTransform = CGAffineTransform(scaleX: 2, y: 2)
let transform3D = CATransform3DMakeAffineTransform(affineTransform)
myLayer.transform = transform3D
```

---

### Сдвиг (Skew) — нестандартная трансформация

```swift
extension CGAffineTransform {
    static func skew(x: CGFloat, y: CGFloat) -> CGAffineTransform {
        return CGAffineTransform(a: 1, b: y, c: x, d: 1, tx: 0, ty: 0)
    }
}

// Использование
let skewTransform = CGAffineTransform.skew(x: 0.5, y: 0)
imageView.transform = skewTransform
```

---

### Сброс трансформации

```swift
// Анимация возврата в исходное состояние
UIView.animate(withDuration: 0.3) {
    view.transform = .identity
}
```

---

### Лучшие практики

1.  **Используйте `CGAffineTransform.identity` для сброса:** Всегда начинайте комбинацию с `identity`.
2.  **Порядок трансформаций имеет значение:** `translate` после `rotate` даёт другой результат, чем до.
3.  **Для сложных трансформаций используйте `concatenating`:** Это эффективнее цепочки вызовов.
4.  **При работе с жестами используйте текущую трансформацию как основу:**

```swift
// Правильно
imageView.transform = imageView.transform.scaledBy(x: scale, y: scale)

// Неправильно (сбрасывает предыдущие трансформации)
imageView.transform = CGAffineTransform(scaleX: scale, y: scale)
```

5.  **Анимируйте трансформации в `UIView.animate`:** Core Animation автоматически интерполирует `CGAffineTransform`.

---

### Итог

**`CGAffineTransform`** — это фундаментальный инструмент для 2D-трансформаций в iOS. Он позволяет:

1.  **Перемещать, масштабировать, вращать и сдвигать** элементы.
2.  **Комбинировать трансформации** в любой последовательности.
3.  **Анимировать** изменения плавно.
4.  **Обрабатывать жесты** (pinch, rotate, pan) в реальном времени.
5.  **Работать с Core Graphics** и `CALayer`.

Понимание `CGAffineTransform` необходимо для создания интерактивных и анимированных интерфейсов, обработки жестов и работы с графикой .