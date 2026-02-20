**UILabel** — это подкласс **[[UIView]]**, предназначенный исключительно для **отображения текста** в iOS-приложениях (и в macOS через [[AppKit]] как NSTextField в некоторых случаях).

Это один из самых часто используемых элементов интерфейса в [[UIKit]] — практически в каждом экране есть хотя бы один UILabel.

В 2026 году UILabel остаётся **незаменимым** в UIKit-проектах, несмотря на появление `Text` в [[SwiftUI]], потому что:
- даёт **полный контроль** над стилем, атрибутами и производительностью,
- идеально работает с [[Auto Layout]], Dynamic Type, тёмной темой,
- поддерживает сложный атрибутированный текст,
- используется в ячейках таблиц/коллекций, заголовках, подписях и т.д.

### Основные характеристики UILabel (актуально на 2026 год)

| Свойство / Возможность                 | Значение по умолчанию     | Что делает / зачем нужен                                          | Самый частый сценарий                                   |
| -------------------------------------- | ------------------------- | ----------------------------------------------------------------- | ------------------------------------------------------- |
| `text`                                 | [[nil]]                   | Обычный текст ([[String]])                                        | Простые надписи                                         |
| `attributedText`                       | `nil`                     | Текст с атрибутами (цвет, шрифт, подчёркивание и т.д.)            | Богато оформленный текст                                |
| `font`                                 | `.systemFont(ofSize: 17)` | Шрифт и размер                                                    | `.preferredFont(forTextStyle:)` — стандарт 2026         |
| `textColor`                            | `.label`                  | Цвет текста (адаптируется к тёмной теме)                          | `.label`, `.secondaryLabel`                             |
| `textAlignment`                        | `.natural`                | Выравнивание (.left, .center, .right, .justified)                 | `.center` для заголовков                                |
| `numberOfLines`                        | `1`                       | Максимальное количество строк (0 = неограниченно)                 | `0` для многострочного текста                           |
| `lineBreakMode`                        | `.byTruncatingTail`       | Как обрезается текст (.byWordWrapping, .byTruncatingMiddle и др.) | `.byWordWrapping` + `numberOfLines = 0`                 |
| `adjustsFontSizeToFitWidth`            | `false`                   | Автоматическое уменьшение шрифта, чтобы текст уместился           | `true` + `minimumScaleFactor` для однострочных надписей |
| `minimumScaleFactor`                   | `0`                       | Минимальный масштаб шрифта при `adjustsFontSizeToFitWidth`        | `0.5` — популярное значение                             |
| `isUserInteractionEnabled`             | `false`                   | Разрешает ли UILabel получать касания                             | Включают для тапов по тексту                            |
| `baselineAdjustment`                   | `.alignCenters`           | Как выравнивается текст по базовой линии при уменьшении           | Редко меняют                                            |
| `allowsDefaultTighteningForTruncation` | `false`                   | Автоматическое сжатие межсимвольных интервалов при обрезке        | `true` для длинных заголовков                           |

### Самые популярные contentMode для UILabel

| contentMode                  | Эффект на UILabel (редко используется) | Когда применять |
|------------------------------|----------------------------------------|------------------|
| `.scaleToFill`               | Растягивает текст (искажает)           | Почти никогда |
| `.scaleAspectFit`            | Вписывает текст, сохраняя пропорции    | Редко |
| `.scaleAspectFill`           | Заполняет, обрезая текст               | Редко |
| `.center`                    | Центрирует без масштабирования         | Почти всегда (по умолчанию) |

**Важно**:  
`contentMode` почти **не влияет** на UILabel — он работает только с изображениями внутри. Для текста используйте `textAlignment`, `numberOfLines`, `lineBreakMode`.

### Самые популярные и рекомендуемые паттерны 2026 года

#### 1. Базовое использование (самый частый)

```swift
let label = UILabel()
label.text = "Привет, мир!"
label.textColor = .label
label.font = .preferredFont(forTextStyle: .title1)  // адаптивный Dynamic Type
label.textAlignment = .center
label.numberOfLines = 0
label.adjustsFontForContentSizeCategory = true  // важно для Dynamic Type
view.addSubview(label)
```

#### 2. Атрибутированный текст (самый мощный сценарий)

```swift
let fullText = "Добро пожаловать в приложение!\nЭто ваш первый вход."
let attributed = NSMutableAttributedString(string: fullText)

attributed.addAttributes([
    .font: UIFont.boldSystemFont(ofSize: 24),
    .foregroundColor: UIColor.systemBlue
], range: NSRange(location: 0, length: 22))

attributed.addAttributes([
    .font: UIFont.preferredFont(forTextStyle: .body),
    .foregroundColor: UIColor.secondaryLabel
], range: NSRange(location: 23, length: fullText.count - 23))

label.attributedText = attributed
label.numberOfLines = 0
label.textAlignment = .center
```

#### 3. Многострочный текст с обрезкой и многоточием

```swift
label.text = "Очень длинный текст, который должен обрезаться красиво с многоточием в конце."
label.numberOfLines = 2
label.lineBreakMode = .byTruncatingTail
```

#### 4. Автоматическое уменьшение шрифта для однострочных надписей

```swift
label.text = "Очень длинное название продукта или заголовок"
label.numberOfLines = 1
label.adjustsFontSizeToFitWidth = true
label.minimumScaleFactor = 0.5
label.baselineAdjustment = .alignCenters
```

#### 5. UILabel как кнопка (с тапом)

```swift
label.isUserInteractionEnabled = true
label.addGestureRecognizer(UITapGestureRecognizer(target: self, action: #selector(labelTapped)))

@objc private func labelTapped() {
    print("Текст нажат")
    // открываем ссылку, показываем алерт и т.д.
}
```

### Лучшие практики UILabel в Swift 2026

- **Всегда** используйте `.preferredFont(forTextStyle:)` — это обеспечивает поддержку Dynamic Type и масштабирование текста  
- **Включайте** `adjustsFontForContentSizeCategory = true` — критично для доступности  
- **Для многострочного текста** — `numberOfLines = 0` + `lineBreakMode = .byWordWrapping`  
- **Для атрибутированного текста** — используйте `NSMutableAttributedString` — это даёт полный контроль  
- **Для скругления и теней** — применяйте `layer.cornerRadius`, `clipsToBounds`, `layer.shadow...`  
- **Для [[SwiftUI]]** — используйте `Text` — UILabel нужен только в смешанных проектах через [[UIViewRepresentable]]  
- **Для производительности** — избегайте большого количества атрибутированного текста в таблицах/коллекциях — лучше кэшировать [[NSAttributedString]]  
- **Документируйте** — пишите комментарий «UILabel — заголовок с Dynamic Type и центрированием, поддержка тёмной темы»

**Короткий итог 2026**:
> UILabel — это **специализированный UIView** для отображения текста (одно- и многострочного).  
> В 2026 году:  
> - ключевые свойства — `text`, `attributedText`, `font` (preferredFont), `textColor` (.label), `numberOfLines`  
> - для адаптивности — `.preferredFont(forTextStyle:)` + `adjustsFontForContentSizeCategory = true`  
> - для стилизации — `attributedText` + `NSMutableAttributedString`  
> - это **самый базовый** и **самый часто встречающийся** элемент для текста в UIKit-приложениях.
