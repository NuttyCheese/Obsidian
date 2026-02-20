**UIEdgeInsets** — это структура в [[UIKit]], которая описывает **отступы** (insets) со всех четырёх сторон прямоугольника: сверху, слева, снизу и справа.

Это один из самых часто используемых типов в [[iOS]]-разработке — он применяется практически везде, где нужно задать отступы, поля, внутренние/внешние отступы, безопасные зоны и т.д.

### Структура UIEdgeInsets

```swift
public struct UIEdgeInsets {
    public var top:    CGFloat
    public var left:   CGFloat
    public var bottom: CGFloat
    public var right:  CGFloat
    
    public init(top: CGFloat, left: CGFloat, bottom: CGFloat, right: CGFloat)
}
```

Есть также удобные статические значения и инициализаторы:

```swift
let zero      = UIEdgeInsets.zero                  // все 0
let symmetric = UIEdgeInsets(top: 16, left: 20, bottom: 16, right: 20)
let horizontalOnly = UIEdgeInsets(horizontal: 24)  // top = bottom = 0, left = right = 24 (iOS 16+)
let verticalOnly   = UIEdgeInsets(vertical: 32)    // left = right = 0, top = bottom = 32   (iOS 16+)
```

### Самые популярные и часто используемые сценарии (2026 актуально)

| Где используется                                  | Пример кода                                                                                               | Что делает / зачем нужен                                       |
| ------------------------------------------------- | --------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| contentInset в [[UIScrollView]] / [[UITableView]] | `tableView.contentInset = UIEdgeInsets(top: 16, left: 0, bottom: 100, right: 0)`                          | Отступы внутри скролла (под навбар, над таббаром, под кнопкой) |
| layoutMargins / directionalLayoutMargins          | `view.directionalLayoutMargins = NSDirectionalEdgeInsets(top: 20, leading: 16, bottom: 20, trailing: 16)` | Автоматические отступы для [[Auto Layout]] (учитывает RTL)     |
| safeAreaInsets (чтение)                           | `let bottomInset = view.safeAreaInsets.bottom`                                                            | Получение высоты home indicator / Dynamic Island               |
| cell layoutMargins                                | `cell.layoutMargins = UIEdgeInsets(top: 8, left: 16, bottom: 8, right: 16)`                               | Отступы содержимого ячейки от границ                           |
| [[UIButton]] contentEdgeInsets                    | `button.contentEdgeInsets = UIEdgeInsets(top: 12, left: 20, bottom: 12, right: 20)`                       | Внутренние отступы текста/иконки внутри кнопки                 |
| [[UIView]] padding / container insets             | `stackView.layoutMargins = UIEdgeInsets(all: 16)`                                                         | Отступы внутри стека или контейнера                            |
| [[UINavigationBar]] / [[UIToolbar]] padding       | `navigationBar.layoutMargins = UIEdgeInsets(horizontal: 16)`                                              | Отступы элементов в навбаре                                    |
| [[UICollectionView]] section insets               | `section.contentInsets = NSDirectionalEdgeInsets(all: 16)`                                                | Отступы секции в compositional layout                          |

### Полезные расширения (очень популярны в 2026)

```swift
extension UIEdgeInsets {
    static func all(_ value: CGFloat) -> UIEdgeInsets {
        UIEdgeInsets(top: value, left: value, bottom: value, right: value)
    }
    
    static func horizontal(_ value: CGFloat) -> UIEdgeInsets {
        UIEdgeInsets(top: 0, left: value, bottom: 0, right: value)
    }
    
    static func vertical(_ value: CGFloat) -> UIEdgeInsets {
        UIEdgeInsets(top: value, left: 0, bottom: value, right: 0)
    }
    
    var totalHorizontal: CGFloat { left + right }
    var totalVertical:   CGFloat { top + bottom }
    
    func adding(top: CGFloat = 0, left: CGFloat = 0, bottom: CGFloat = 0, right: CGFloat = 0) -> UIEdgeInsets {
        UIEdgeInsets(
            top:    self.top    + top,
            left:   self.left   + left,
            bottom: self.bottom + bottom,
            right:  self.right  + right
        )
    }
}
```

Использование:

```swift
tableView.contentInset = .all(16)                       // отступ 16 pt со всех сторон
button.contentEdgeInsets = .horizontal(20)              // только слева и справа
stackView.directionalLayoutMargins = .vertical(24).adding(left: 16)
```

### NSDirectionalEdgeInsets — современный брат UIEdgeInsets

С iOS 11+ появился `NSDirectionalEdgeInsets` — версия, которая учитывает **направление чтения** (LTR / RTL).

```swift
let insets = NSDirectionalEdgeInsets(top: 16, leading: 24, bottom: 16, trailing: 24)

// Автоматически меняет leading/trailing при смене языка на арабский/иврит
view.directionalLayoutMargins = insets
```

**Правило 2026**:  
Если вы работаете с layoutMargins, contentInsets, sectionInsets → **всегда используйте NSDirectionalEdgeInsets**.

### Лучшие практики UIEdgeInsets в Swift 2026

- **Для layoutMargins / contentInsets** — используйте **[[NSDirectionalEdgeInsets]]** (учитывает RTL)  
- **Для contentInset в скроллах** — используйте **UIEdgeInsets** (они не directional)  
- **Для кнопок** — `contentEdgeInsets`, `titleEdgeInsets`, `imageEdgeInsets` — всё в UIEdgeInsets  
- **Для compositional layout** — `contentInsets`, `interGroupSpacing` — в NSDirectionalEdgeInsets  
- **Для кастомных отступов** — добавьте расширения `.all`, `.horizontal`, `.vertical`  
- **Для safe area** — никогда не хардкодьте значения — всегда берите `view.safeAreaInsets`  
- **Документируйте** — пишите комментарий «UIEdgeInsets — отступы контента таблицы: сверху 16 pt (под навбар), снизу 100 pt (под кнопкой)»

**Короткий итог 2026**:
> UIEdgeInsets — это **структура отступов** со всех четырёх сторон (top, left, bottom, right).  
> В 2026 году:  
> - для layoutMargins, sectionInsets → используйте **NSDirectionalEdgeInsets**  
> - для contentInset скроллов — **UIEdgeInsets**  
> - самый частый паттерн — `.all(16)`, `.horizontal(20)`, `.vertical(24)`  
> - это **самый базовый** и **самый часто встречающийся** тип для любых отступов в UIKit  
