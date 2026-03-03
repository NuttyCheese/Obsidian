#extension #uiview 

---
## 1. Управление взаимодействием (User Interaction)

# [[UIKit]] — Полезные расширения для [[UIView]] ([[Swift]])

Коллекция самых часто используемых методов-расширений для `UIView`, которые сильно упрощают и ускоряют написание интерфейса в UIKit.

Все методы написаны с поддержкой **method chaining** (`@discardableResult → Self`), где это уместно.

## 1. Взаимодействие с пользователем

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

**Типичные сценарии**
- Отключение всего экрана на время загрузки / анимации
- Заморозка кнопок/ячеек во время редактирования таблицы/коллекции
- Временное блокирование при показе модальных окон, алертов, side menu

```swift
// Пример
view
    .disableUserInteraction()
    // ... запуск анимации / сети ...
    .enableUserInteraction()   // в конце
```

## 2. Подготовка к [[Auto Layout]] и добавление дочерних вью

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
            $0.disableAutoresizingMask()
            addSubview($0)
        }
        return self
    }
}
```

**Когда какой метод удобнее**

| Метод                                   | Когда использовать                                         |
| --------------------------------------- | ---------------------------------------------------------- |
| `addSubviews(label, button, image)`     | 2–7 элементов прямо в коде                                 |
| `addSubviews(arrayOfViews)`             | элементы уже собраны в массив ([[map]], [[filter]] и т.д.) |
| `addSubviewsAndDisableAutoresizingMask` | **самый частый** сценарий при создании UI программно       |

**Пример типичного `setupUI()`**

```swift
private func setupUI() {
    let stack = UIStackView()
        .disableAutoresizingMask()
        .axis(.vertical)
        .spacing(16)
        .alignment(.fill)
    
    let title    = UILabel().text("Профиль").disableAutoresizingMask()
    let avatar   = UIImageView().disableAutoresizingMask()
    let button   = UIButton(type: .system).title("Сохранить").disableAutoresizingMask()
    
    stack.addSubviews(title, avatar, button)
    
    view.addSubview(stack)
    
    stack.pinToSafeArea(insets: .init(top: 20, left: 16, bottom: 20, right: 16))
}
```

## 3. Закругление углов и границы

```swift
extension UIView {
    /// Закругляет все углы (рекомендуемый способ)
    @discardableResult
    func roundAllCorners(radius: CGFloat) -> Self {
        layer.cornerRadius = radius
        layer.masksToBounds = true
        return self
    }
    
    /// Закругляет выбранные углы через маску (устаревший, но иногда нужный подход)
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

**Сравнение способов закругления углов (2025–2026)**

| Способ                            | Выборочные углы           | Тень сохраняется         | Производительность | Offscreen rendering | Рекомендация            |
| --------------------------------- | ------------------------- | ------------------------ | ------------------ | ------------------- | ----------------------- |
| `layer.cornerRadius`              | Нет / через maskedCorners | Да (если !masksToBounds) | Отличная           | Нет                 | **Основной выбор**      |
| `roundCorners` ([[CAShapeLayer]]) | Да                        | Нет                      | Средняя–плохая     | Да                  | Только если очень нужно |

## 4. Удобные методы для констрейнтов

```swift
extension UIView {
    /// Привязывает ко всем сторонам superview (с insets)
    func pinToSuperview(insets: UIEdgeInsets = .zero) {
        guard let superview else { return }
        disableAutoresizingMask()
        
        NSLayoutConstraint.activate([
            topAnchor.constraint(equalTo: superview.topAnchor, constant: insets.top),
            leftAnchor.constraint(equalTo: superview.leftAnchor, constant: insets.left),
            rightAnchor.constraint(equalTo: superview.rightAnchor, constant: -insets.right),
            bottomAnchor.constraint(equalTo: superview.bottomAnchor, constant: -insets.bottom)
        ])
    }
    
    /// Привязывает к safe area superview
    func pinToSafeArea(insets: UIEdgeInsets = .zero) {
        guard let superview else { return }
        disableAutoresizingMask()
        
        NSLayoutConstraint.activate([
            topAnchor.constraint(equalTo: superview.safeAreaLayoutGuide.topAnchor, constant: insets.top),
            leftAnchor.constraint(equalTo: superview.safeAreaLayoutGuide.leftAnchor, constant: insets.left),
            rightAnchor.constraint(equalTo: superview.safeAreaLayoutGuide.rightAnchor, constant: -insets.right),
            bottomAnchor.constraint(equalTo: superview.safeAreaLayoutGuide.bottomAnchor, constant: -insets.bottom)
        ])
    }
    
    /// Центрирует внутри superview
    func centerInSuperview() {
        guard let superview else { return }
        disableAutoresizingMask()
        
        NSLayoutConstraint.activate([
            centerXAnchor.constraint(equalTo: superview.centerXAnchor),
            centerYAnchor.constraint(equalTo: superview.centerYAnchor)
        ])
    }
    
    /// Устанавливает фиксированный размер
    func setSize(width: CGFloat? = nil, height: CGFloat? = nil) {
        disableAutoresizingMask()
        
        var constraints: [NSLayoutConstraint] = []
        
        if let width { constraints.append(widthAnchor.constraint(equalToConstant: width)) }
        if let height { constraints.append(heightAnchor.constraint(equalToConstant: height)) }
        
        NSLayoutConstraint.activate(constraints)
    }
}
```

**Примеры использования констрейнтов**

```swift
// Полноэкранная вью
imageView.pinToSafeArea()

// Кнопка по центру с отступами
button
    .centerInSuperview()
    .setSize(width: 200, height: 48)

// Карточка с отступами 16 со всех сторон
cardView.pinToSuperview(insets: .init(all: 16))
```

---

