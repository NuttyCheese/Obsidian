## 📘 Определение

**`NSLayoutConstraint`** — объект в [[UIKit]], который описывает **ограничение (constraint) для Auto Layout**.  
Позволяет программно задавать **позицию и размер элементов интерфейса** относительно других элементов или супервью.  
Относится к **UIKit → Auto Layout**.

---

## 🔹 Примеры кода

### 1. Создание простого [[constraint]] для ширины

```swift
import UIKit

let view = UIView()
view.translatesAutoresizingMaskIntoConstraints = false
let widthConstraint = NSLayoutConstraint(item: view,
                                         attribute: .width,
                                         relatedBy: .equal,
                                         toItem: nil,
                                         attribute: .notAnAttribute,
                                         multiplier: 1.0,
                                         constant: 100)
view.addConstraint(widthConstraint)
```

---

### 2. Constraint для выравнивания по центру супервью

```swift
let superview = UIView()
let subview = UIView()
superview.addSubview(subview)
subview.translatesAutoresizingMaskIntoConstraints = false

NSLayoutConstraint.activate([
    subview.centerXAnchor.constraint(equalTo: superview.centerXAnchor),
    subview.centerYAnchor.constraint(equalTo: superview.centerYAnchor)
])
```

---

### 3. Constraint с привязкой к другому элементу

```swift
let label = UILabel()
let button = UIButton()
superview.addSubview(label)
superview.addSubview(button)

label.translatesAutoresizingMaskIntoConstraints = false
button.translatesAutoresizingMaskIntoConstraints = false

NSLayoutConstraint.activate([
    label.topAnchor.constraint(equalTo: superview.topAnchor, constant: 20),
    label.leadingAnchor.constraint(equalTo: superview.leadingAnchor, constant: 20),
    button.topAnchor.constraint(equalTo: label.bottomAnchor, constant: 10),
    button.leadingAnchor.constraint(equalTo: label.leadingAnchor)
])
```

---

### 4. Constraint для высоты и ширины

```swift
let squareView = UIView()
superview.addSubview(squareView)
squareView.translatesAutoresizingMaskIntoConstraints = false

NSLayoutConstraint.activate([
    squareView.widthAnchor.constraint(equalToConstant: 100),
    squareView.heightAnchor.constraint(equalToConstant: 100)
])
```

---

### 5. Использование мультипликатора

```swift
let parentView = UIView()
let childView = UIView()
parentView.addSubview(childView)
childView.translatesAutoresizingMaskIntoConstraints = false

NSLayoutConstraint.activate([
    childView.widthAnchor.constraint(equalTo: parentView.widthAnchor, multiplier: 0.5),
    childView.heightAnchor.constraint(equalTo: parentView.heightAnchor, multiplier: 0.3)
])
```

---

### 6. Изменение существующего constraint

```swift
let heightConstraint = squareView.heightAnchor.constraint(equalToConstant: 50)
heightConstraint.isActive = true

// Изменяем значение
heightConstraint.constant = 100
```
