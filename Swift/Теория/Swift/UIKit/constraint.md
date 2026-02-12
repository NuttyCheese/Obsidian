#uikit #Swift #ios 
## 📘 Определение

**Constraint** — ограничение в Auto Layout, описывающее **отношения между элементами интерфейса** или между элементом и его родителем.  
Позволяет задать размер, положение или соотношение элементов, чтобы интерфейс корректно масштабировался на разных устройствах.  
В [[Swift]] используется через **[[UIKit]] → [[NSLayoutConstraint]]**, **[[SnapKit]]**, **[[SwiftUI]] → modifiers** (`.frame`, `.layoutPriority`, `.alignmentGuide`).

---

## 🔹 Примеры кода

### 1. Простое ограничение ширины и высоты через NSLayoutConstraint

```swift
let view = UIView()
view.translatesAutoresizingMaskIntoConstraints = false
NSLayoutConstraint.activate([
    view.widthAnchor.constraint(equalToConstant: 100),
    view.heightAnchor.constraint(equalToConstant: 50)
])
```

---

### 2. Ограничение привязки к родительскому элементу

```swift
let parentView = UIView()
let childView = UIView()
parentView.addSubview(childView)
childView.translatesAutoresizingMaskIntoConstraints = false

NSLayoutConstraint.activate([
    childView.topAnchor.constraint(equalTo: parentView.topAnchor, constant: 10),
    childView.leadingAnchor.constraint(equalTo: parentView.leadingAnchor, constant: 10),
    childView.trailingAnchor.constraint(equalTo: parentView.trailingAnchor, constant: -10),
    childView.bottomAnchor.constraint(equalTo: parentView.bottomAnchor, constant: -10)
])
```

---

### 3. Ограничение соотношения сторон

```swift
let box = UIView()
box.translatesAutoresizingMaskIntoConstraints = false

NSLayoutConstraint.activate([
    box.widthAnchor.constraint(equalTo: box.heightAnchor, multiplier: 1.0) // квадрат
])
```

---

### 4. Ограничения через SnapKit

```swift
import SnapKit

let view = UIView()
parentView.addSubview(view)

view.snp.makeConstraints { make in
    make.top.equalTo(parentView).offset(20)
    make.left.equalTo(parentView).offset(20)
    make.right.equalTo(parentView).offset(-20)
    make.height.equalTo(50)
}
```

---

### 5. Ограничения с приоритетами

```swift
let label = UILabel()
label.translatesAutoresizingMaskIntoConstraints = false

let constraint = label.widthAnchor.constraint(equalToConstant: 200)
constraint.priority = .defaultLow
constraint.isActive = true
```

---

### 6. [[SwiftUI]] — аналог constraint

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        Text("Hello")
            .frame(width: 100, height: 50)
            .padding()
    }
}
```
