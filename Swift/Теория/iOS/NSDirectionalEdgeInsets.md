**NSDirectionalEdgeInsets** — это структура в [[UIKit]] (появилась в iOS 11, 2017 год), которая описывает **отступы** (insets) со всех четырёх сторон прямоугольника, но с учётом **направления чтения** (left-to-right / right-to-left).

Это **эволюция** старого [[UIEdgeInsets]], адаптированная под поддержку языков с письмом справа налево (арабский, иврит, фарси и др.).

### Основная разница между UIEdgeInsets и NSDirectionalEdgeInsets

| Свойство                     | UIEdgeInsets (старый)                  | NSDirectionalEdgeInsets (новый)                     | Когда использовать в 2026 году |
|------------------------------|----------------------------------------|-----------------------------------------------------|--------------------------------|
| top                          | всегда верх                            | всегда верх                                         | одинаково |
| left                         | всегда слева                           | **leading** (слева в LTR, справа в RTL)             | NSDirectional — предпочтительнее |
| bottom                       | всегда низ                             | всегда низ                                          | одинаково |
| right                        | всегда справа                          | **trailing** (справа в LTR, слева в RTL)            | NSDirectional — предпочтительнее |
| Автоматическая адаптация под RTL | Нет                                    | Да (автоматически меняет leading/trailing)          | NSDirectional — must-have |

**Коротко**:  
`left` → `leading`  
`right` → `trailing`  
`top` и `bottom` остаются неизменными.

### Основные свойства NSDirectionalEdgeInsets

```swift
public struct NSDirectionalEdgeInsets {
    public var top:      CGFloat
    public var leading:  CGFloat   // ← левая сторона в LTR, правая в RTL
    public var bottom:   CGFloat
    public var trailing: CGFloat   // ← правая сторона в LTR, левая в RTL
    
    public init(top: CGFloat, leading: CGFloat, bottom: CGFloat, trailing: CGFloat)
}
```

Удобные инициализаторы и значения (iOS 16+):

```swift
let zero      = NSDirectionalEdgeInsets.zero
let all16     = NSDirectionalEdgeInsets(all: 16)          // все стороны 16 pt
let horizontal24 = NSDirectionalEdgeInsets(horizontal: 24) // leading = trailing = 24
let vertical32   = NSDirectionalEdgeInsets(vertical: 32)   // top = bottom = 32
```

### Самые популярные и рекомендуемые паттерны в 2026 году

#### 1. Layout Margins (самый частый случай)

```swift
// Рекомендуемый способ в 2026 — directional
view.directionalLayoutMargins = NSDirectionalEdgeInsets(
    top: 16,
    leading: 20,
    bottom: 16,
    trailing: 20
)

// Автоматически адаптируется под арабский / иврит
```

#### 2. Content Insets в [[UITableView]] / [[UICollectionView]]

```swift
tableView.contentInset = UIEdgeInsets(top: 16, left: 0, bottom: 100, right: 0)
// ↑ для contentInset всё ещё UIEdgeInsets (не directional)

// Но для секций в compositional layout — directional!
let section = NSCollectionLayoutSection(group: group)
section.contentInsets = NSDirectionalEdgeInsets(top: 8, leading: 16, bottom: 8, trailing: 16)
```

#### 3. [[UIStackView]] layoutMargins

```swift
stackView.isLayoutMarginsRelativeArrangement = true
stackView.directionalLayoutMargins = NSDirectionalEdgeInsets(all: 16)
```

#### 4. Кастомный отступ в [[UIButton]] / [[UILabel]]

```swift
button.contentEdgeInsets = UIEdgeInsets(top: 12, left: 20, bottom: 12, right: 20)
// ↑ contentEdgeInsets всё ещё UIEdgeInsets

// Но если нужно directional поведение — используйте directionalLayoutMargins
// или кастомные constraints с leading/trailing
```

### Лучшие практики NSDirectionalEdgeInsets в Swift 2026

- **Всегда** используйте **NSDirectionalEdgeInsets** вместо `UIEdgeInsets`, когда работаете с:
  - `directionalLayoutMargins`
  - `contentInsets` в `NSCollectionLayoutSection`
  - `layoutMargins` в `UIStackView`
  - `directionalEdgeInsets` в `UICollectionViewFlowLayout` (если используете flow layout)
- **Для contentInset в UIScrollView / UITableView / UICollectionView** — всё ещё `UIEdgeInsets` (они **не directional**)
- **Для кнопок** — `contentEdgeInsets`, `titleEdgeInsets`, `imageEdgeInsets` — остаются `UIEdgeInsets`
- **Для Auto Layout** — используйте `leading` / `trailing` вместо `left` / `right` в constraints
- **Для RTL-языков** — тестируйте приложение с арабским / ивритом — directional insets автоматически переворачиваются
- **Документируйте** — пишите комментарий «NSDirectionalEdgeInsets — отступы секции коллекции с поддержкой RTL (leading/trailing)»

**Короткий итог 2026**:
> NSDirectionalEdgeInsets — это **RTL-дружественная** версия UIEdgeInsets.  
> В 2026 году:  
> - используйте **везде**, где есть `directionalLayoutMargins`, `contentInsets` в compositional layout, `directional` свойства  
> - `leading` = левая сторона в LTR / правая в RTL  
> - `trailing` = правая сторона в LTR / левая в RTL  
> - `top` и `bottom` — не меняются  
> - для contentInset скроллов — всё ещё `UIEdgeInsets`  
> Это **современный стандарт** для создания приложений, которые выглядят правильно на всех языках и направлениях письма.
