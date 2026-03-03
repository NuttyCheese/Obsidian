#extension #uiview 

---
# [[UIKit]] — полезные расширения для [[UIView]]

Коллекция часто используемых [[extension]]'ов для `UIView`, которые упрощают повседневную работу с интерфейсом в UIKit.

## 1. Управление взаимодействием (User Interaction)

```swift
extension UIView {
    func enableUserInteraction() {
        isUserInteractionEnabled = true
    }
    
    func disableUserInteraction() {
        isUserInteractionEnabled = false
    }
}
```

### Зачем нужны эти методы

- Значительно лучше читаемость кода
- Поддерживают цепочки (method chaining)
- Симметричная пара enable / disable

### Примеры использования

```swift
// Отключить всё на время загрузки
view.disableUserInteraction()
overlayView.isHidden = false

// Вернуть взаимодействие
view.enableUserInteraction()
overlayView.isHidden = true
```

### Вариант с chaining (самый популярный)

```swift
extension UIView {
    @discardableResult
    func enableUserInteraction() -> Self {
        isUserInteractionEnabled = true
        return self
    }
    
    @discardableResult
    func disableUserInteraction() -> Self {
        isUserInteractionEnabled = false
        return self
    }
}
```

```swift
imageView
    .enableUserInteraction()
    .addGesture(tapGesture)
```

## 2. Подготовка к [[Auto Layout ]]и добавление subviews

```swift
extension UIView {
    /// Отключает autoresizing mask (обязательно для ручного Auto Layout)
    @discardableResult
    func disableAutoresizingMask() -> Self {
        translatesAutoresizingMaskIntoConstraints = false
        return self
    }
    
    /// Добавляет несколько subviews (variadic)
    @discardableResult
    func addSubviews(_ views: UIView...) -> Self {
        views.forEach { addSubview($0) }
        return self
    }
    
    /// Добавляет subviews из массива
    @discardableResult
    func addSubviews(_ views: [UIView]) -> Self {
        views.forEach { addSubview($0) }
        return self
    }
    
    /// Самый популярный комбинированный метод
    @discardableResult
    func addSubviewsAndDisableAutoresizingMask(_ views: UIView...) -> Self {
        views.forEach {
            $0.translatesAutoresizingMaskIntoConstraints = false
            addSubview($0)
        }
        return self
    }
}
```

### Когда какой метод использовать

| Метод                                   | Когда удобно                                        |
| --------------------------------------- | --------------------------------------------------- |
| `addSubviews(label, button)`            | 2–6 элементов прямо в коде                          |
| `addSubviews(array)`                    | элементы уже в массиве ([[map]], [[filter]] и т.д.) |
| `addSubviewsAndDisableAutoresizingMask` | самый частый сценарий при программном UI            |

### Пример типичного setupUI()

```swift
private func setupUI() {
    let stack = UIStackView()
        .disableAutoresizingMask()
        .withAxis(.vertical)
        .withSpacing(12)
    
    let title    = UILabel().withText("Настройки").disableAutoresizingMask()
    let subtitle = UILabel().withText("v3.2.1").disableAutoresizingMask()
    let button   = UIButton(type: .system).withTitle("Сохранить").disableAutoresizingMask()
    
    stack.addSubviews(title, subtitle, button)
    
    view.addSubview(stack)
    
    NSLayoutConstraint.activate([
        stack.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 24),
        stack.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
        stack.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20)
    ])
}
```

## 3. Закругление углов и границы

```swift
extension UIView {
    
    /// Закругляет **все** углы (самый быстрый и рекомендуемый способ)
    @discardableResult
    func roundAllCorners(radius: CGFloat) -> Self {
        layer.cornerRadius = radius
        layer.masksToBounds = true
        return self
    }
    
    /// Закругляет **выбранные** углы через маску (устаревший подход)
    func roundCorners(_ corners: UIRectCorner, radius: CGFloat) {
        if bounds == .zero {
            DispatchQueue.main.asyncAfter(deadline: .now()) { [weak self] in
                self?.roundCorners(corners, radius: radius)
            }
            return
        }
        
        let path = UIBezierPath(
            roundedRect: bounds,
            byRoundingCorners: corners,
            cornerRadii: CGSize(width: radius, height: radius)
        )
        
        let mask = CAShapeLayer()
        mask.path = path.cgPath
        mask.frame = bounds
        layer.mask = mask
    }
    
    /// Добавляет границу
    @discardableResult
    func addBorder(color: UIColor, width: CGFloat) -> Self {
        layer.borderColor = color.cgColor
        layer.borderWidth = width
        return self
    }
}
```

### Сравнение подходов к закруглению углов (2025–2026)

| Метод                  | Выборочные углы | Тень сохраняется | Производительность | Offscreen rendering | Рекомендация сейчас              |
|------------------------|------------------|-------------------|---------------------|-----------------------|----------------------------------|
| `layer.cornerRadius`   | Нет (или через maskedCorners) | Да (если !masksToBounds) | Отличная          | Обычно нет           | **Основной выбор**               |
| `roundCorners` (mask)  | Да               | Нет               | Средняя / плохая   | Да                   | Только legacy или < iOS 11       |
| `addBorder`            | —                | —                 | Отличная           | Нет                  | Использовать с cornerRadius      |

### Современный стиль карточки (рекомендуемый 2025+)

```swift
class ModernCardView: UIView {
    override func layoutSubviews() {
        super.layoutSubviews()
        
        layer.cornerRadius = 20
        layer.maskedCorners = [.layerMinXMinYCorner, .layerMaxXMinYCorner]
        
        layer.borderWidth = 1
        layer.borderColor = UIColor.systemGray4.cgColor
        
        layer.shadowColor = UIColor.black.cgColor
        layer.shadowOpacity = 0.12
        layer.shadowRadius = 10
        layer.shadowOffset = .zero
        
        // важно для производительности в коллекциях/таблицах
        layer.shadowPath = UIBezierPath(roundedRect: bounds, cornerRadius: 20).cgPath
    }
}
```

---
