#bug #error #ui #autolayout 

---

Это предупреждение появляется, когда **набор Auto Layout ограничений (constraints) противоречит друг другу**.

- [[iOS]] пытается удовлетворить все constraints, но некоторые из них несовместимы → система вынуждена **игнорировать часть ограничений**.
    
- Результат: неправильное расположение UI-элементов, сжатие, растяжение или неожиданные размеры.
    

Причины появления:

1. Несовместимые размеры или позиции для одного view.
    
2. Дублирующие constraints (например, height + top + bottom, которые не совместимы).
    
3. Constraints с одинаковым приоритетом, которые не могут быть одновременно выполнены.
    
4. Добавление constraints программно и через Interface Builder без синхронизации.
    

---

### Примеры кода/сценариев возникновения

**Пример 1: Несовместимые height и top-bottom**

```swift
let view = UIView()
view.translatesAutoresizingMaskIntoConstraints = false
NSLayoutConstraint.activate([
    view.topAnchor.constraint(equalTo: superview.topAnchor),
    view.bottomAnchor.constraint(equalTo: superview.bottomAnchor),
    view.heightAnchor.constraint(equalToConstant: 100) // ⚠️ конфликт с top-bottom
])
```

- Высота фиксирована 100, но top и bottom хотят растянуть view по высоте superview → конфликт.
    

---

**Пример 2: Дублирующие width constraints**

```swift
let view = UIView()
view.translatesAutoresizingMaskIntoConstraints = false
NSLayoutConstraint.activate([
    view.widthAnchor.constraint(equalToConstant: 50),
    view.widthAnchor.constraint(equalToConstant: 60) // ⚠️ конфликт
])
```

- Одновременно нельзя установить width = 50 и width = 60 → система игнорирует одно из ограничений.
    

---

**Пример 3: Constraints с одинаковым приоритетом**

```swift
let view = UIView()
view.translatesAutoresizingMaskIntoConstraints = false
NSLayoutConstraint.activate([
    view.leadingAnchor.constraint(equalTo: superview.leadingAnchor),
    view.trailingAnchor.constraint(equalTo: superview.trailingAnchor),
    view.centerXAnchor.constraint(equalTo: superview.centerXAnchor) // ⚠️ потенциальный конфликт
])
```

- Несовместимые горизонтальные constraints → Auto Layout выберет одно, выдаст предупреждение.
    

---

### Как исправить

#### 1️⃣ Проверять приоритеты

```swift
let heightConstraint = view.heightAnchor.constraint(equalToConstant: 100)
heightConstraint.priority = .defaultHigh
heightConstraint.isActive = true
```

- Устанавливаем приоритет, чтобы система знала, какое ограничение можно пожертвовать.
    

---

#### 2️⃣ Проверять дублирующие constraints

- Удаляем лишние constraints в коде или Interface Builder.
    
- Используем `NSLayoutConstraint.deactivate([...])` перед добавлением новых constraints.
    

---

#### 3️⃣ Использовать StackView

- [[UIStackView]] автоматически управляет constraints для дочерних view → меньше конфликтов.
    

---

#### 4️⃣ Отладка с [[UIView]] методами

- `view.constraints` и `view.debugDescription` показывают текущие активные constraints.
    
- В консоли [[Xcode]] читаем предупреждение `"Unable to simultaneously satisfy constraints"` → смотрим на conflicting constraints, указанные в сообщении.
    

---

### Пример исправления

**Было:**

```swift
view.topAnchor.constraint(equalTo: superview.topAnchor).isActive = true
view.bottomAnchor.constraint(equalTo: superview.bottomAnchor).isActive = true
view.heightAnchor.constraint(equalToConstant: 100).isActive = true
```

- Конфликт → предупреждение.
    

**Исправлено:**

```swift
let heightConstraint = view.heightAnchor.constraint(equalToConstant: 100)
heightConstraint.priority = .defaultHigh
NSLayoutConstraint.activate([
    view.topAnchor.constraint(equalTo: superview.topAnchor),
    view.bottomAnchor.constraint(equalTo: superview.bottomAnchor),
    heightConstraint
])
```

- Теперь система может пожертвовать высотой при необходимости → предупреждение исчезает.
    

---
