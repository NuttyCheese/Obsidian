**NSLayoutYAxisAnchor** — это специальный класс в **[[Auto Layout]]** ([[UIKit]]), который представляет **вертикальную ось** (Y-ось) представления и позволяет создавать **ограничения** ([[constraint]]s) по вертикали: верхняя граница (top), нижняя (bottom), центр по Y, базовая линия текста и т.д.

Он наследуется от **NSLayoutAnchor** и используется исключительно для **вертикальных** ограничений.

### Основные методы и свойства NSLayoutYAxisAnchor

| Метод / свойство                             | Что создаёт ограничение на                       | Пример использования (современный стиль 2026)                                                      |
| -------------------------------------------- | ------------------------------------------------ | -------------------------------------------------------------------------------------------------- |
| `topAnchor`                                  | верхняя граница                                  | `view.topAnchor.constraint(equalTo: superview.safeAreaLayoutGuide.topAnchor, constant: 16)`        |
| `bottomAnchor`                               | нижняя граница                                   | `view.bottomAnchor.constraint(equalTo: superview.safeAreaLayoutGuide.bottomAnchor, constant: -16)` |
| `centerYAnchor`                              | центр по вертикали                               | `view.centerYAnchor.constraint(equalTo: superview.centerYAnchor)`                                  |
| `firstBaselineAnchor`                        | первая базовая линия текста (для UILabel и т.п.) | `label.firstBaselineAnchor.constraint(equalTo: otherLabel.firstBaselineAnchor)`                    |
| `lastBaselineAnchor`                         | последняя базовая линия текста                   | `label.lastBaselineAnchor.constraint(equalTo: otherLabel.lastBaselineAnchor)`                      |
| `constraint(equalTo: multiplier: constant:)` | пропорциональное ограничение                     | `view.heightAnchor.constraint(equalTo: superview.heightAnchor, multiplier: 0.5)`                   |
| `constraint(equalToConstant:)`               | фиксированная высота                             | `view.heightAnchor.constraint(equalToConstant: 44)`                                                |
| `constraint(greaterThanOrEqualToConstant:)`  | минимальная высота                               | `view.heightAnchor.constraint(greaterThanOrEqualToConstant: 100)`                                  |
| `constraint(lessThanOrEqualToConstant:)`     | максимальная высота                              | `view.heightAnchor.constraint(lessThanOrEqualToConstant: 300)`                                     |

### Самые популярные и рекомендуемые паттерны 2026

#### 1. Стандартное размещение с отступами от safe area (самый частый)

```swift
view.translatesAutoresizingMaskIntoConstraints = false

NSLayoutConstraint.activate([
    view.topAnchor.constraint(equalTo: view.superview!.safeAreaLayoutGuide.topAnchor, constant: 16),
    view.bottomAnchor.constraint(equalTo: view.superview!.safeAreaLayoutGuide.bottomAnchor, constant: -16),
    view.leadingAnchor.constraint(equalTo: view.superview!.safeAreaLayoutGuide.leadingAnchor, constant: 16),
    view.trailingAnchor.constraint(equalTo: view.superview!.safeAreaLayoutGuide.trailingAnchor, constant: -16)
])
```

#### 2. Центрирование по вертикали

```swift
NSLayoutConstraint.activate([
    button.centerYAnchor.constraint(equalTo: view.centerYAnchor),
    button.centerXAnchor.constraint(equalTo: view.centerXAnchor),
    button.widthAnchor.constraint(equalToConstant: 200),
    button.heightAnchor.constraint(equalToConstant: 44)
])
```

#### 3. Пропорциональная высота относительно ширины (aspect ratio)

```swift
imageView.translatesAutoresizingMaskIntoConstraints = false

NSLayoutConstraint.activate([
    imageView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20),
    imageView.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
    imageView.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16),
    
    // Высота = ширина * 9/16 (вертикальное видео)
    imageView.heightAnchor.constraint(equalTo: imageView.widthAnchor, multiplier: 9.0/16.0)
])
```

#### 4. Выравнивание базовых линий текста (для [[UILabel]], [[UITextField]])

```swift
titleLabel.translatesAutoresizingMaskIntoConstraints = false
subtitleLabel.translatesAutoresizingMaskIntoConstraints = false

NSLayoutConstraint.activate([
    titleLabel.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20),
    titleLabel.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
    
    subtitleLabel.firstBaselineAnchor.constraint(equalTo: titleLabel.lastBaselineAnchor, constant: 8),
    subtitleLabel.leadingAnchor.constraint(equalTo: titleLabel.leadingAnchor)
])
```

#### 5. Минимальная и максимальная высота

```swift
textView.translatesAutoresizingMaskIntoConstraints = false

NSLayoutConstraint.activate([
    textView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20),
    textView.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
    textView.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16),
    
    // Высота от 100 до 300
    textView.heightAnchor.constraint(greaterThanOrEqualToConstant: 100),
    textView.heightAnchor.constraint(lessThanOrEqualToConstant: 300)
])
```

### 6. Лучшие практики NSLayoutYAxisAnchor в 2026

- **Всегда** используй **[[safeAreaLayoutGuide]]** для отступов от краёв экрана (особенно top и bottom)  
- **Предпочитай** `topAnchor` / `bottomAnchor` для вертикальных отступов  
- **Используй** `firstBaselineAnchor` / `lastBaselineAnchor` при выравнивании текста в разных UILabel  
- **Не меняй** `constant` внутри `layoutSubviews()` — это может вызвать layout cycle  
- **Для пропорций** — используй `multiplier` — самый гибкий способ  
- **Активируй** сразу массивом через `NSLayoutConstraint.activate([…])` — чисто и эффективно  
- **В [[SwiftUI]]** — почти никогда не нужен (GeometryReader, .frame, .offset, .aspectRatio)  
- **Документируйте** — пиши комментарий «top = safeArea.top + 20 (стандартный отступ от верха)»

**Короткий девиз 2026**:
> NSLayoutYAxisAnchor — это когда ты говоришь: «по вертикали я хочу быть вот так относительно другого представления».  
> В 2026 году:  
> - `top` / `bottom` — всегда с safeAreaLayoutGuide  
> - `centerYAnchor` — для центрирования  
> - `multiplier` — для aspect ratio и пропорций  
> - `firstBaseline` / `lastBaseline` — для выравнивания текста  
> - activate массивом — самый чистый стиль  
> Это **основа** вертикально адаптивного и читаемого Auto Layout в UIKit.
