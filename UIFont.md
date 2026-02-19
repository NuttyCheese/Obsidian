**`UIFont`** — это класс в [[UIKit]], который представляет **шрифт** для отображения текста в iOS-приложениях.

Он отвечает за:
- выбор семейства шрифта (font family),
- размер,
- стиль (обычный, жирный, курсив и т.д.),
- динамический тип (Dynamic Type — адаптация размера текста под настройки системы),
- метрики шрифта (размеры, высота строки и т.д.).

### Основные способы создания UIFont (актуально на 2026 год)

| Способ создания                               | Пример кода                                                                 | Когда использовать в 2026 |
|-----------------------------------------------|-----------------------------------------------------------------------------|----------------------------|
| Системный шрифт (самый частый)                | `UIFont.systemFont(ofSize: 17)`<br>`UIFont.systemFont(ofSize: 17, weight: .semibold)` | Почти всегда — основной выбор |
| Жирный системный шрифт                        | `UIFont.boldSystemFont(ofSize: 17)`                                         | Заголовки, кнопки          |
| Курсивный системный шрифт                     | `UIFont.italicSystemFont(ofSize: 17)`                                       | Редко (лучше .medium/.semibold) |
| С Dynamic Type (адаптивный размер)            | `UIFont.preferredFont(forTextStyle: .body)`<br>`UIFont.preferredFont(forTextStyle: .title1, compatibleWith: traitCollection)` | **Обязательно** для UI-текста |
| Кастомный шрифт из ресурсов приложения        | `UIFont(name: "HelveticaNeue-Bold", size: 17)`                              | Кастомные шрифты (.ttf/.otf) |
| Шрифт с фиксированным размером (игнорирует Dynamic Type) | `UIFont.systemFont(ofSize: 17)`                                             | Только для иконок, фиксированных UI-элементов |

### Самые популярные и рекомендуемые паттерны UIFont в 2026

#### 1. Dynamic Type — золотой стандарт (обязательно в [[SwiftUI]]/UIKit 2026)

```swift
// Лучший способ для всего текста в UI
label.font = UIFont.preferredFont(forTextStyle: .body)

// Варианты textStyle (от маленького к большому)
.body, .callout, .subheadline, .footnote, .caption1, .caption2
.title3, .title2, .title1, .largeTitle, .headline
```

#### 2. Полный контроль над стилем + весом

```swift
// Полужирный текст для заголовков
let headlineFont = UIFont.systemFont(ofSize: 17, weight: .semibold)

// Жирный + Dynamic Type
let dynamicBold = UIFontMetrics(forTextStyle: .headline).scaledFont(
    for: UIFont.systemFont(ofSize: 17, weight: .bold)
)
```

#### 3. Кастомный шрифт из проекта (с Dynamic Type)

```swift
extension UIFont {
    static func customFont(name: String, textStyle: UIFont.TextStyle) -> UIFont {
        let baseFont = UIFont(name: name, size: 17) ?? UIFont.systemFont(ofSize: 17)
        return UIFontMetrics(forTextStyle: textStyle).scaledFont(for: baseFont)
    }
}

// Использование
label.font = .customFont(name: "SFProDisplay-Regular", textStyle: .body)
```

#### 4. Получение метрик шрифта (очень полезно для кастомных layout)

```swift
let font = UIFont.preferredFont(forTextStyle: .body)
let lineHeight = font.lineHeight
let ascent = font.ascender
let descent = font.descender
let leading = font.leading

// Для точного расчёта высоты строки
let paragraphStyle = NSMutableParagraphStyle()
paragraphStyle.lineSpacing = 4
let attributes: [NSAttributedString.Key: Any] = [
    .font: font,
    .paragraphStyle: paragraphStyle
]
```

### Лучшие практики UIFont в Swift 2026

- **Всегда** используйте `preferredFont(forTextStyle:)` для пользовательского текста — это автоматически уважает настройки Dynamic Type пользователя  
- **Никогда** не используйте фиксированные размеры (`size: 17`) для текста, который читает пользователь  
- **Для кнопок/иконок/фиксированных UI-элементов** — фиксированный размер допустим  
- **Для кастомных шрифтов** — всегда добавляйте Dynamic Type поддержку через `UIFontMetrics`  
- **В SwiftUI** — предпочтительнее `.font(.body)`, `.font(.title)` и т.д. — они используют те же text styles  
- **В UIKit** — применяйте `UIFontMetrics` для масштабирования кастомных шрифтов  
- **Swift 6 strict concurrency** — `UIFont` полностью безопасен ([[Sendable]])  
- **Документируйте** — пишите комментарий «UIFont.preferredFont(forTextStyle: .body) — адаптивный системный шрифт с учётом Dynamic Type»

**Короткий итог 2026**:
> `UIFont` — это **шрифт для UIKit** с полной поддержкой **Dynamic Type** и [[Unicode]].  
> В 2026 году:  
> - основной способ — `preferredFont(forTextStyle:)`  
> - фиксированные размеры — только для иконок/фиксированных UI  
> - кастомные шрифты — обязательно с `UIFontMetrics`  
> - это **основа** доступного и адаптивного текста в iOS-приложениях  
