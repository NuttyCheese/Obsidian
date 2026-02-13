## 📘 Определение

**SnapKit** — это библиотека для [[iOS]], которая позволяет **создавать Auto Layout констрейнты программно** с использованием **DSL (Domain Specific Language)**.  
Упрощает работу с **[[NSLayoutConstraint]]**, делая код более читаемым и компактным.  
Относится к **[[UIKit]] → Auto Layout**.

---

## 🔹 Примеры кода

### 1. Установка констрейнтов для [[Swift/Расширения/UIView]]

```swift
import SnapKit
import UIKit

let superview = UIView()
let subview = UIView()
superview.addSubview(subview)

subview.snp.makeConstraints { make in
    make.width.height.equalTo(100)  // ширина и высота 100
    make.center.equalToSuperview()   // по центру супервью
}
```

---

### 2. Привязка к другому элементу

```swift
let label = UILabel()
let button = UIButton()
superview.addSubview(label)
superview.addSubview(button)

label.snp.makeConstraints { make in
    make.top.equalToSuperview().offset(20)
    make.leading.equalToSuperview().offset(20)
}

button.snp.makeConstraints { make in
    make.top.equalTo(label.snp.bottom).offset(10)
    make.leading.equalTo(label)
}
```

---

### 3. Использование мультипликатора

```swift
subview.snp.makeConstraints { make in
    make.width.equalTo(superview.snp.width).multipliedBy(0.5)
    make.height.equalTo(superview.snp.height).multipliedBy(0.3)
}
```

---

### 4. Установка отступов с `edges`

```swift
subview.snp.makeConstraints { make in
    make.edges.equalToSuperview().inset(20) // отступы 20 со всех сторон
}
```

---

### 5. Обновление констрейнтов

```swift
var heightConstraint: Constraint?

subview.snp.makeConstraints { make in
    heightConstraint = make.height.equalTo(50).constraint
}

// позже можно обновить
heightConstraint?.update(offset: 100)
```
