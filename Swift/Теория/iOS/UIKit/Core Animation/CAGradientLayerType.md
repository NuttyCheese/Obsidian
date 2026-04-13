#core-animation #cagradientlayer #gradient #animation #ios #swift

---

## CAGradientLayerType — Типы градиентов в Core Animation

### Определение

**`CAGradientLayerType`** — это перечисление ([[enum]]) в [[Core Animation]], которое определяет **тип градиента** для [[CAGradientLayer]]. Оно позволяет выбрать способ интерполяции цветов между начальной и конечной точками: линейный (axial), радиальный (radial) или коноидальный (conic).

По умолчанию `CAGradientLayer` использует линейный градиент (`.axial`). Понимание всех трёх типов открывает широкие возможности для создания разнообразных визуальных эффектов.

### Зачем это знать iOS-разработчику?

1.  **Разнообразие эффектов:** Линейные, радиальные и коноидальные градиенты создают разные визуальные впечатления.
2.  **Анимация:** Все типы градиентов можно анимировать.
3.  **Современные интерфейсы:** Коноидальные градиенты используются в неоморфизме и современных UI-трендах.
4.  **Эффекты глубины:** Радиальные градиенты создают эффект "сияния" или "линзы".
5.  **Производительность:** Все три типа отрисовываются на [[GPU]].

---

### Три типа градиента

| Тип | Значение | Описание |
|---|---|---|
| **`.axial`** | `"axial"` | Линейный градиент (по умолчанию). Цвета изменяются вдоль прямой линии от `startPoint` к `endPoint`. |
| **`.radial`** | `"radial"` | Радиальный градиент. Цвета изменяются от центральной точки (`center`) к внешнему радиусу (`endRadius`). |
| **`.conic`** | `"conic"` | Коноидальный (конический) градиент. Цвета изменяются вокруг центральной точки (`center`), как в цветовом круге. |

---

### Синтаксис

```swift
let gradientLayer = CAGradientLayer()
gradientLayer.type = .axial      // линейный (по умолчанию)
gradientLayer.type = .radial     // радиальный
gradientLayer.type = .conic      // коноидальный
```

---

## 1. Axial Gradient (Линейный градиент)

### Описание

Цвета интерполируются вдоль прямой линии между `startPoint` и `endPoint`. Это самый распространённый тип градиента.

### Свойства

| Свойство         | Тип         | Описание                           |
| ---------------- | ----------- | ---------------------------------- |
| **`startPoint`** | [[CGPoint]] | Начальная точка градиента (0...1). |
| **`endPoint`**   | `CGPoint`   | Конечная точка градиента (0...1).  |

### Примеры

#### Базовый линейный градиент

```swift
import UIKit

class AxialGradientViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let gradientLayer = CAGradientLayer()
        gradientLayer.frame = view.bounds
        gradientLayer.type = .axial
        gradientLayer.colors = [UIColor.red.cgColor, UIColor.blue.cgColor]
        gradientLayer.startPoint = CGPoint(x: 0, y: 0.5)
        gradientLayer.endPoint = CGPoint(x: 1, y: 0.5)
        
        view.layer.addSublayer(gradientLayer)
    }
}
```

#### Горизонтальный градиент

```swift
gradientLayer.startPoint = CGPoint(x: 0, y: 0.5)
gradientLayer.endPoint = CGPoint(x: 1, y: 0.5)
```

#### Вертикальный градиент

```swift
gradientLayer.startPoint = CGPoint(x: 0.5, y: 0)
gradientLayer.endPoint = CGPoint(x: 0.5, y: 1)
```

#### Диагональный градиент

```swift
gradientLayer.startPoint = CGPoint(x: 0, y: 0)
gradientLayer.endPoint = CGPoint(x: 1, y: 1)
```

---

## 2. Radial Gradient (Радиальный градиент)

### Описание

Цвета интерполируются от центральной точки к внешнему радиусу. Создаёт эффект "линзы", "сияния" или "круглого перехода".

### Свойства

| Свойство          | Тип         | Описание                                         |
| ----------------- | ----------- | ------------------------------------------------ |
| **`center`**      | `CGPoint`   | Центр градиента (0...1).                         |
| **`startRadius`** | [[CGFloat]] | Начальный радиус (обычно 0).                     |
| **`endRadius`**   | `CGFloat`   | Конечный радиус (обычно половина ширины/высоты). |

### Примеры

#### Базовый радиальный градиент

```swift
class RadialGradientViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let gradientLayer = CAGradientLayer()
        gradientLayer.frame = view.bounds
        gradientLayer.type = .radial
        gradientLayer.colors = [UIColor.yellow.cgColor, UIColor.orange.cgColor, UIColor.red.cgColor]
        gradientLayer.center = CGPoint(x: 0.5, y: 0.5)
        gradientLayer.startRadius = 0
        gradientLayer.endRadius = view.bounds.width / 2
        
        view.layer.addSublayer(gradientLayer)
    }
}
```

#### Эффект "линзы"

```swift
let lensGradient = CAGradientLayer()
lensGradient.frame = view.bounds
lensGradient.type = .radial
lensGradient.colors = [UIColor.white.cgColor, UIColor.clear.cgColor]
lensGradient.center = CGPoint(x: 0.5, y: 0.5)
lensGradient.startRadius = 0
lensGradient.endRadius = 100
```

#### Эффект "прожектора"

```swift
let spotlight = CAGradientLayer()
spotlight.frame = view.bounds
spotlight.type = .radial
spotlight.colors = [UIColor.white.withAlphaComponent(0.8).cgColor,
                    UIColor.clear.cgColor]
spotlight.center = CGPoint(x: 0.3, y: 0.3)  // Смещённый центр
spotlight.startRadius = 0
spotlight.endRadius = 150
```

---

## 3. Conic Gradient (Коноидальный градиент)

### Описание

Цвета интерполируются вокруг центральной точки, как в цветовом круге. Создаёт эффект "радуги" или "спирали".

### Свойства

| Свойство | Тип | Описание |
|---|---|---|
| **`center`** | `CGPoint` | Центр градиента (0...1). |

### Примеры

#### Базовый коноидальный градиент

```swift
class ConicGradientViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let gradientLayer = CAGradientLayer()
        gradientLayer.frame = view.bounds
        gradientLayer.type = .conic
        gradientLayer.colors = [
            UIColor.red.cgColor,
            UIColor.yellow.cgColor,
            UIColor.green.cgColor,
            UIColor.blue.cgColor,
            UIColor.purple.cgColor,
            UIColor.red.cgColor
        ]
        gradientLayer.center = CGPoint(x: 0.5, y: 0.5)
        
        view.layer.addSublayer(gradientLayer)
    }
}
```

#### Цветовой круг

```swift
let colorWheel = CAGradientLayer()
colorWheel.frame = view.bounds
colorWheel.type = .conic
colorWheel.colors = [
    UIColor(red: 1, green: 0, blue: 0, alpha: 1).cgColor,      // 0°
    UIColor(red: 1, green: 1, blue: 0, alpha: 1).cgColor,      // 60°
    UIColor(red: 0, green: 1, blue: 0, alpha: 1).cgColor,      // 120°
    UIColor(red: 0, green: 1, blue: 1, alpha: 1).cgColor,      // 180°
    UIColor(red: 0, green: 0, blue: 1, alpha: 1).cgColor,      // 240°
    UIColor(red: 1, green: 0, blue: 1, alpha: 1).cgColor,      // 300°
    UIColor(red: 1, green: 0, blue: 0, alpha: 1).cgColor       // 360°
]
colorWheel.center = CGPoint(x: 0.5, y: 0.5)
```

#### Градиент с прозрачностью

```swift
let transparentConic = CAGradientLayer()
transparentConic.frame = view.bounds
transparentConic.type = .conic
transparentConic.colors = [
    UIColor.blue.cgColor,
    UIColor.blue.withAlphaComponent(0).cgColor,
    UIColor.green.cgColor,
    UIColor.green.withAlphaComponent(0).cgColor,
    UIColor.red.cgColor
]
transparentConic.center = CGPoint(x: 0.5, y: 0.5)
```

---

### Анимация разных типов градиентов

#### Анимация цветов (для любого типа)

```swift
func animateGradientColors(layer: CAGradientLayer) {
    let animation = CABasicAnimation(keyPath: "colors")
    animation.fromValue = [UIColor.red.cgColor, UIColor.blue.cgColor]
    animation.toValue = [UIColor.green.cgColor, UIColor.yellow.cgColor]
    animation.duration = 2.0
    animation.autoreverses = true
    animation.repeatCount = .infinity
    
    layer.add(animation, forKey: "colorChange")
}
```

#### Анимация радиального градиента (изменение радиуса)

```swift
func animateRadialRadius(layer: CAGradientLayer) {
    let animation = CABasicAnimation(keyPath: "endRadius")
    animation.fromValue = 0
    animation.toValue = 200
    animation.duration = 1.5
    animation.autoreverses = true
    animation.repeatCount = .infinity
    
    layer.add(animation, forKey: "radiusChange")
}
```

#### Анимация коноидального градиента (вращение)

```swift
func animateConicRotation(layer: CAGradientLayer) {
    let animation = CABasicAnimation(keyPath: "transform.rotation.z")
    animation.fromValue = 0
    animation.toValue = CGFloat.pi * 2
    animation.duration = 4
    animation.repeatCount = .infinity
    
    layer.add(animation, forKey: "rotation")
}
```

---

### Сравнение типов

| Характеристика | Axial | Radial | Conic |
|---|---|---|---|
| **Форма** | Линия | Круг | Круг |
| **Параметры** | startPoint, endPoint | center, startRadius, endRadius | center |
| **Типичное применение** | Фоны, переходы | Сияния, линзы, прожекторы | Цветовые круги, радуга |
| **Анимация цвета** | ✅ | ✅ | ✅ |
| **Анимация позиции** | ✅ (startPoint/endPoint) | ✅ (center, radius) | ✅ (center, rotation) |
| **Сложность** | Низкая | Средняя | Средняя |

---

### Комбинирование типов

```swift
class CombinedGradientsView: UIView {
    override func layoutSubviews() {
        super.layoutSubviews()
        
        // Фоновый линейный градиент
        let backgroundGradient = CAGradientLayer()
        backgroundGradient.frame = bounds
        backgroundGradient.type = .axial
        backgroundGradient.colors = [UIColor.darkGray.cgColor, UIColor.black.cgColor]
        backgroundGradient.startPoint = CGPoint(x: 0, y: 0)
        backgroundGradient.endPoint = CGPoint(x: 1, y: 1)
        layer.addSublayer(backgroundGradient)
        
        // Радиальное свечение в центре
        let glowGradient = CAGradientLayer()
        glowGradient.frame = bounds
        glowGradient.type = .radial
        glowGradient.colors = [UIColor.yellow.withAlphaComponent(0.3).cgColor,
                               UIColor.clear.cgColor]
        glowGradient.center = CGPoint(x: 0.5, y: 0.5)
        glowGradient.startRadius = 0
        glowGradient.endRadius = 150
        layer.addSublayer(glowGradient)
        
        // Коноидальный градиент поверх (эффект радуги)
        let conicGradient = CAGradientLayer()
        conicGradient.frame = bounds
        conicGradient.type = .conic
        conicGradient.colors = [
            UIColor.red.withAlphaComponent(0.3).cgColor,
            UIColor.blue.withAlphaComponent(0.3).cgColor
        ]
        conicGradient.center = CGPoint(x: 0.5, y: 0.5)
        layer.addSublayer(conicGradient)
    }
}
```

---

### Лучшие практики

1.  **По умолчанию используйте `.axial`** — он самый простой и производительный.
2.  **Для эффектов свечения используйте `.radial`.**
3.  **Для цветовых кругов и современных UI используйте `.conic`.**
4.  **Всегда задавайте `frame` перед добавлением градиента.**
5.  **Анимируйте через `CABasicAnimation` для плавности.**

```swift
// Правильно: задаём frame
gradientLayer.frame = view.bounds
view.layer.addSublayer(gradientLayer)

// Неправильно: добавили до задания frame
view.layer.addSublayer(gradientLayer)
gradientLayer.frame = view.bounds  // Не обновится автоматически
```

---

### Итог

**`CAGradientLayerType`** — это перечисление, определяющее три типа градиентов в Core Animation:

1.  **`.axial`** (линейный) — классический, простой, подходит для большинства случаев.
2.  **`.radial`** (радиальный) — круговой переход, эффекты свечения, линзы.
3.  **`.conic`** (коноидальный) — цветовой круг, радуга, современные UI-эффекты.

Понимание этих типов позволяет создавать разнообразные визуальные эффекты — от простых фонов до сложных анимированных градиентов .