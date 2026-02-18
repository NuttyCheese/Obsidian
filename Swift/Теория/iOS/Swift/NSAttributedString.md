**`NSAttributedString`** — это **неизменяемый** ([[immutable]]) объект в [[Foundation]], который представляет **текст вместе с набором атрибутов форматирования**.  
Он позволяет отображать текст с разными стилями (цвет, шрифт, подчёркивание, зачёркивание, межстрочный интервал, выравнивание и т.д.) в одном объекте.

> Проще говоря:  
> Обычная [[String]] — это просто текст.  
> `NSAttributedString` — это текст + инструкции, как его красить, подчёркивать и стилизовать.

### 1. Когда и зачем используют NSAttributedString в 2025–2026

| Сценарий (реальный)                               | Почему именно NSAttributedString                         | Альтернатива (когда можно обойтись без него)   |
| ------------------------------------------------- | -------------------------------------------------------- | ---------------------------------------------- |
| Разноцветный текст в [[UILabel]] / [[UITextView]] | Нужны разные цвета, шрифты, подчёркивания в одном лейбле | `AttributedString` ([[SwiftUI]] / [[iOS]] 15+) |
| Кликабельные ссылки в тексте                      | `link` атрибут + `UITextView` [[delegate]]               | Markdown в [[UILabel]] (iOS 15+)               |
| Подчёркивание / зачёркивание текста               | `.underlineStyle`, `.strikethroughStyle`                 | Markdown или `AttributedString`                |
| Разные шрифты в одном лейбле (жирный + обычный)   | `.font` для разных диапазонов                            | `AttributedString`                             |
| Форматирование цены / скидки                      | "1000 ₽" → "1000" обычный, "₽" серый                     | `AttributedString`                             |
| Выделение эмодзи / символов                       | `.font` + `.foregroundColor` для отдельных символов      | —                                              |

### 2. Ключевые отличия: NSAttributedString vs NSMutableAttributedString

| Тип                                      | Изменяемость | Когда использовать в 2026                              |
|------------------------------------------|--------------|--------------------------------------------------------|
| `NSAttributedString`                     | Неизменяемый | Финальная версия текста, которую отдаёшь в UI          |
| `NSMutableAttributedString`              | Изменяемый   | Постепенное построение (добавление атрибутов по частям) |

**Правило 2026**:  
- Строишь текст → используй `NSMutableAttributedString`  
- Готово → присвой `label.attributedText = mutable` (автоматически преобразуется в immutable)

### 3. Самый популярный паттерн 2026 (сокращённая запись + chaining)

```swift
let text = NSMutableAttributedString(string: "Привет, ")

text.append(
    NSAttributedString(
        string: "мир!",
        attributes: [
            .foregroundColor: UIColor.systemBlue,
            .font: UIFont.boldSystemFont(ofSize: 20),
            .underlineStyle: NSUnderlineStyle.single.rawValue
        ]
    )
)

// Или более современно (iOS 15+ стиль)
let attrString = NSMutableAttributedString()
    .appending("Привет, ", attributes: [.foregroundColor: UIColor.gray])
    .appending("мир!", attributes: [
        .foregroundColor: UIColor.systemBlue,
        .font: UIFont.boldSystemFont(ofSize: 20),
        .underlineStyle: NSUnderlineStyle.single.rawValue
    ])

label.attributedText = attrString
```

### 4. Самые используемые атрибуты (топ-2026)

| Ключ (NSAttributedString.Key) | Тип значения                          | Пример использования                                |
| ----------------------------- | ------------------------------------- | --------------------------------------------------- |
| `.font`                       | [[UIFont]]                            | `.systemFont(ofSize: 16, weight: .semibold)`        |
| `.foregroundColor`            | [[UIColor]] / `NSColor`               | `.systemBlue`, `.label`                             |
| `.backgroundColor`            | `UIColor`                             | Выделение текста цветом фона                        |
| `.underlineStyle`             | `NSUnderlineStyle` rawValue ([[Int]]) | `.single`, `.double`, `.thick`                      |
| `.strikethroughStyle`         | `NSUnderlineStyle` rawValue           | Зачёркивание цены                                   |
| `.link`                       | [[URL]]                               | Кликабельные ссылки в `UITextView`                  |
| `.paragraphStyle`             | `NSParagraphStyle`                    | Выравнивание, межстрочный интервал, отступы         |
| `.kern`                       | `NSNumber` ([[CGFloat]])              | Межбуквенный интервал                               |
| `.baselineOffset`             | `NSNumber` (CGFloat)                  | Смещение по вертикали (для верхних/нижних индексов) |

### 5. Лучшие практики NSAttributedString в 2026

- **Строить через `NSMutableAttributedString`** → добавлять части текста методом `.append(...)`  
- **Использовать chaining** — выглядит современно и читаемо  
- **Для кликабельных ссылок** — всегда используй `.link` + `UITextView.delegate`  
- **Для SwiftUI** → переходи на `AttributedString` (iOS 15+), он полностью Swift-native  
- **Не создавай NSAttributedString в цикле** — это дорого, лучше собирай один mutable объект  
- **Swift 6 strict concurrency** — `NSAttributedString` полностью `Sendable` и безопасен  
- **Документируйте** — пиши комментарий «NSAttributedString — форматированный текст с подчёркиванием цены»

**Короткий девиз 2026**:
> `NSAttributedString` — это когда обычный текст становится **красивым**: цвет, шрифт, подчёркивание, ссылки — всё в одном объекте.  
> В 2026 году:  
> - строй через `NSMutableAttributedString` + `.append`  
> - для UI — присваивай `attributedText`  
> - для кликабельных ссылок — `.link` + `UITextView`  
> - в новом коде → рассмотри `AttributedString` (Swift-native)  
> Это **основа** красивого и функционального текста в UIKit.

Удачи с ярким и стильным текстом в твоём приложении! 🎨