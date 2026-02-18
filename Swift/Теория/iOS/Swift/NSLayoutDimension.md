**NSLayoutDimension** — это специальный класс в **[[Auto Layout]]** ([[UIKit]]), который представляет **одно измерение** (ширину или высоту) представления и позволяет создавать **ограничения ([[constraint]])** относительно других измерений.

Он используется для создания **пропорциональных**, **относительных** или **фиксированных** ограничений ширины/высоты.

### Основные возможности NSLayoutDimension

| Метод / свойство                          | Что делает                                                                 | Пример использования (2025–2026 стиль) |
|-------------------------------------------|-----------------------------------------------------------------------------|----------------------------------------|
| `constraint(equalTo:)`                    | Равно другому измерению                                                    | `widthAnchor.constraint(equalTo: other.widthAnchor)` |
| `constraint(equalToConstant:)`            | Фиксированная константа                                                    | `heightAnchor.constraint(equalToConstant: 44)` |
| `constraint(equalTo: multiplier:)`        | Пропорциональное отношение (самое частое)                                  | `widthAnchor.constraint(equalTo: superview.widthAnchor, multiplier: 0.8)` |
| `constraint(equalTo: multiplier: constant:)` | Пропорция + смещение                                                     | `heightAnchor.constraint(equalTo: widthAnchor, multiplier: 16/9, constant: 0)` |
| `constraint(greaterThanOrEqualToConstant:)` | Минимум (≥)                                                               | `widthAnchor.constraint(greaterThanOrEqualToConstant: 200)` |
| `constraint(lessThanOrEqualToConstant:)`  | Максимум (≤)                                                               | `heightAnchor.constraint(lessThanOrEqualToConstant: 300)` |
| `constant` (после создания)               | Изменение значения ограничения в рантайме                                  | `heightConstraint.constant = 100`      |
| `priority`                                | Приоритет ограничения (от 1 до 1000)                                       | `constraint.priority = .defaultHigh`   |
| `isActive`                                | Активация/деактивация ограничения                                          | `constraint.isActive = true`           |

### Самые популярные и рекомендуемые паттерны 2026

#### 1. Пропорциональная ширина/высота относительно superview (самый частый)

```swift
view.translatesAutoresizingMaskIntoConstraints = false

NSLayoutConstraint.activate([
    view.widthAnchor.constraint(equalTo: view.superview!.widthAnchor, multiplier: 0.7),
    view.heightAnchor.constraint(equalTo: view.widthAnchor, multiplier: 16.0/9.0), // 16:9
    view.centerXAnchor.constraint(equalTo: view.superview!.centerXAnchor),
    view.centerYAnchor.constraint(equalTo: view.superview!.centerYAnchor)
])
```

#### 2. Аспектное соотношение (aspect ratio) — золотой стандарт для изображений/видео

```swift
imageView.translatesAutoresizingMaskIntoConstraints = false

let aspectConstraint = imageView.heightAnchor.constraint(
    equalTo: imageView.widthAnchor,
    multiplier: 4.0/3.0   // 4:3
)
aspectConstraint.priority = .defaultHigh

NSLayoutConstraint.activate([
    imageView.widthAnchor.constraint(equalTo: view.widthAnchor, multiplier: 0.9),
    aspectConstraint,
    imageView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
    imageView.centerYAnchor.constraint(equalTo: view.centerYAnchor)
])
```

#### 3. Минимальная и максимальная ширина/высота

```swift
button.translatesAutoresizingMaskIntoConstraints = false

NSLayoutConstraint.activate([
    button.widthAnchor.constraint(greaterThanOrEqualToConstant: 120),   // минимум 120
    button.widthAnchor.constraint(lessThanOrEqualToConstant: 300),      // максимум 300
    button.heightAnchor.constraint(equalToConstant: 44),
    button.centerXAnchor.constraint(equalTo: view.centerXAnchor),
    button.centerYAnchor.constraint(equalTo: view.centerYAnchor)
])
```

#### 4. Динамическая высота по контенту + максимум

```swift
textView.translatesAutoresizingMaskIntoConstraints = false

let maxHeight: CGFloat = 200

NSLayoutConstraint.activate([
    textView.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
    textView.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16),
    textView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 16),
    
    // Высота по контенту, но не больше maxHeight
    textView.heightAnchor.constraint(lessThanOrEqualToConstant: maxHeight),
    textView.heightAnchor.constraint(greaterThanOrEqualTo: textView.contentSize.heightAnchor)
])
```

### 5. Лучшие практики NSLayoutDimension в 2026

- **Всегда** активируй constraints через `NSLayoutConstraint.activate([…])` — это атомарно и эффективно  
- **Используй multiplier** для пропорций и aspect ratio — это самый гибкий и адаптивный способ  
- **Задавай priority** — особенно при конфликтах (`.required`, `.high`, `.low`)  
- **Не меняй constant** внутри `layoutSubviews()` — это может вызвать layout cycle  
- **Для динамической высоты** — используй `intrinsicContentSize` + `invalidateIntrinsicContentSize()`  
- **В SwiftUI** → почти никогда не нужен NSLayoutDimension (GeometryReader, .frame, .aspectRatio)  
- **Документируйте** — пиши комментарий «height = width * 16/9 (aspect ratio видео)»

**Короткий девиз 2026**:
> NSLayoutDimension — это когда ты говоришь: «ширина/высота должна быть такой-то относительно другого измерения».  
> В 2026 году:  
> - multiplier — твой лучший друг для пропорций и aspect ratio  
> - constant — для фиксированных отступов  
> - greaterThan/lessThan — для min/max размеров  
> - activate сразу массивом — самый чистый стиль  
> Это **основа** адаптивного и красивого Auto Layout в [[UIKit]].
