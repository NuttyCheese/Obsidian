**NSLayoutXAxisAnchor** — это специальный класс в **[[Auto Layout]]** ([[UIKit]]), который представляет **горизонтальную ось** (X-ось) представления и позволяет создавать **ограничения** ([[constraint]]) по горизонтали: ведущий (leading), trailing, центр по X, левая/правая граница и т.д.

Он наследуется от **NSLayoutAnchor** и используется для создания **горизонтальных** ограничений между представлениями.

### Основные методы NSLayoutXAxisAnchor

| Метод / Свойство                              | Что создаёт ограничение на                     | Пример использования (2026 стиль) |
|-----------------------------------------------|------------------------------------------------|-----------------------------------|
| `leadingAnchor`                               | leading (левая граница в LTR, правая в RTL)    | `view.leadingAnchor.constraint(equalTo: superview.leadingAnchor, constant: 16)` |
| `trailingAnchor`                              | trailing (правая граница в LTR, левая в RTL)   | `view.trailingAnchor.constraint(equalTo: superview.trailingAnchor, constant: -16)` |
| `leftAnchor`                                  | левая граница (игнорирует RTL)                 | Редко, только если нужен фиксированный left |
| `rightAnchor`                                 | правая граница (игнорирует RTL)                | Редко, только если нужен фиксированный right |
| `centerXAnchor`                               | центр по горизонтали                           | `view.centerXAnchor.constraint(equalTo: superview.centerXAnchor)` |
| `constraint(equalTo: multiplier: constant:)`  | пропорциональное ограничение                   | `view.leadingAnchor.constraint(equalTo: superview.leadingAnchor, multiplier: 0.5)` |

### Самые популярные и рекомендуемые паттерны 2026

#### 1. Стандартное размещение с отступами (самый частый)

```swift
view.translatesAutoresizingMaskIntoConstraints = false

NSLayoutConstraint.activate([
    view.leadingAnchor.constraint(equalTo: view.superview!.safeAreaLayoutGuide.leadingAnchor, constant: 16),
    view.trailingAnchor.constraint(equalTo: view.superview!.safeAreaLayoutGuide.trailingAnchor, constant: -16),
    view.topAnchor.constraint(equalTo: view.superview!.safeAreaLayoutGuide.topAnchor, constant: 16)
])
```

#### 2. Центрирование по горизонтали

```swift
NSLayoutConstraint.activate([
    button.centerXAnchor.constraint(equalTo: view.centerXAnchor),
    button.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 100),
    button.widthAnchor.constraint(equalToConstant: 200),
    button.heightAnchor.constraint(equalToConstant: 44)
])
```

#### 3. Пропорциональная ширина относительно superview

```swift
NSLayoutConstraint.activate([
    cardView.widthAnchor.constraint(equalTo: view.safeAreaLayoutGuide.widthAnchor, multiplier: 0.85),
    cardView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
    cardView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20)
])
```

#### 4. Фиксированное расстояние между двумя представлениями

```swift
NSLayoutConstraint.activate([
    label.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
    button.leadingAnchor.constraint(equalTo: label.trailingAnchor, constant: 12),
    button.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16)
])
```

#### 5. Адаптивный отступ в зависимости от размера экрана (пропорция)

```swift
let margin = view.safeAreaLayoutGuide.widthAnchor.constraint(equalToConstant: 0)
margin.isActive = true  // будет изменяться позже

// В viewDidLayoutSubviews или traitCollectionDidChange
override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
    super.traitCollectionDidChange(previousTraitCollection)
    
    let multiplier: CGFloat = traitCollection.horizontalSizeClass == .compact ? 0.05 : 0.1
    margin.constant = view.safeAreaLayoutGuide.widthAnchor.multiplier * multiplier
}
```

### 6. Лучшие практики NSLayoutXAxisAnchor в 2026

- **Всегда** используй **[[safeAreaLayoutGuide]]** вместо прямого superview для отступов от краёв экрана  
- **Предпочитай** `leading` / `trailing` над `left` / `right` — это автоматически поддерживает RTL (арабский, иврит)  
- **Не используй** `left` / `right` в новых проектах — только если нужен фиксированный физический край  
- **Активируй** сразу массивом через `NSLayoutConstraint.activate([…])` — это атомарно и эффективно  
- **Не меняй** `constant` внутри `layoutSubviews()` — это может вызвать layout cycle  
- **Для пропорций** — используй `multiplier` — это самый гибкий и адаптивный способ  
- **В [[SwiftUI]]** — почти никогда не нужен NSLayoutXAxisAnchor (GeometryReader, .frame, .offset)  
- **Документируйте** — пиши комментарий «leading = superview.leading + 16 (стандартный отступ)»

**Короткий девиз 2026**:
> NSLayoutXAxisAnchor — это когда ты говоришь: «по горизонтали я хочу быть вот так относительно другого представления».  
> В 2026 году:  
> - `leading` / `trailing` — всегда (поддержка RTL)  
> - `centerXAnchor` — для центрирования  
> - `multiplier` — для пропорций  
> - `constant` — для фиксированных отступов  
> - activate массивом — чистый стиль  
> Это **основа** адаптивного и RTL-friendly Auto Layout в UIKit.
