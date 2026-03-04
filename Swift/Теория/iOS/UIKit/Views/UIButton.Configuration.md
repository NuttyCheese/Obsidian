**UIButton.Configuration** — это современный и рекомендуемый способ создания и настройки кнопок в [[UIKit]], начиная с **[[iOS]] 15** (2021). Это полностью заменяет старый стиль работы с [[UIButton]] (setTitle, setImage, backgroundColor, layer.cornerRadius и т.д.) и даёт:

- **единый объект конфигурации**  
- **автоматическую поддержку состояний** (normal, highlighted, selected, disabled)  
- **встроенные стили** (plain, gray, tinted, filled, bordered, borderedProminent и т.д.)  
- **поддержку SF Symbols с weight/scale**  
- **лучшую доступность** и адаптацию под Dynamic Type  
- **меньше кода** и **более читаемый** результат

В 2026 году это **единственный правильный** способ создавать кнопки в UIKit.

### 1. Основные стили (predefined styles)

| Стиль                     | Внешний вид (светлая/тёмная тема)                     | Когда использовать в 2026 году                          | Пример |
|---------------------------|--------------------------------------------------------|----------------------------------------------------------|--------|
| `.plain`                  | Простой текст, без фона                                | Вторичные действия, ссылки, текстовая кнопка             | Cancel, Learn More |
| `.gray`                   | Серый фон, серый текст                                 | Нейтральные действия, отмена                             | Cancel, Close |
| `.tinted`                 | Фон с акцентным цветом приложения                      | Основные действия, акцентные кнопки                      | Save, Next |
| `.filled`                 | Полностью залитый акцентным цветом                     | Самые важные действия (логин, купить, отправить)         | Submit, Buy Now |
| `.bordered`               | Контур + акцентный текст                               | Второстепенные, но заметные действия                     | Edit, Share |
| `.borderedProminent`      | Контур + залитый фон                                   | Очень важные вторичные действия                          | Upgrade, Sign Up |
| `.borderless` (редко)     | Только текст, без границ                               | В меню, в списках                                        | — |

### 2. Самый современный паттерн 2026 года

#### Вариант 1: Простая кнопка (самый частый)

```swift
let button = UIButton(configuration: .filled(), primaryAction: UIAction { _ in
    print("Кнопка нажата!")
})

button.configuration?.title = "Сохранить"
button.configuration?.baseBackgroundColor = .systemBlue
button.configuration?.baseForegroundColor = .white
button.configuration?.cornerStyle = .medium  // или .large, .capsule, .dynamic
```

#### Вариант 2: Кнопка с иконкой (SF Symbols)

```swift
var config = UIButton.Configuration.filled()
config.title = "Добавить"
config.image = UIImage(systemName: "plus.circle.fill")
config.imagePlacement = .leading          // слева от текста
config.imagePadding = 8                   // отступ между иконкой и текстом
config.cornerStyle = .capsule             // полностью скруглённая

let addButton = UIButton(configuration: config, primaryAction: UIAction { _ in
    print("Добавлено!")
})
```

#### Вариант 3: Кнопка с разными состояниями

```swift
var config = UIButton.Configuration.tinted()
config.title = "Избранное"

// Нормальное состояние
config.baseForegroundColor = .systemBlue

let favoriteButton = UIButton(configuration: config, primaryAction: UIAction { action in
    // Toggle состояния
    action.sender?.isSelected.toggle()
    
    var updatedConfig = action.sender?.configuration
    updatedConfig?.title = action.sender?.isSelected == true ? "В избранном" : "Добавить в избранное"
    updatedConfig?.image = UIImage(systemName: action.sender?.isSelected == true ? "heart.fill" : "heart")
    action.sender?.configuration = updatedConfig
})

// Обновляем конфигурацию при изменении состояния
favoriteButton.configurationUpdateHandler = { button in
    var config = button.configuration ?? UIButton.Configuration.tinted()
    
    if button.isHighlighted {
        config.baseBackgroundColor = config.baseBackgroundColor?.withAlphaComponent(0.7)
    } else if button.isSelected {
        config.baseBackgroundColor = .systemPink.withAlphaComponent(0.2)
        config.baseForegroundColor = .systemPink
    } else {
        config.baseBackgroundColor = nil
        config.baseForegroundColor = .systemBlue
    }
    
    button.configuration = config
}
```

### 3. Полный список самых полезных свойств конфигурации

| Свойство / Метод             | Тип / Значение                              | Что делает / Рекомендация 2026   |
| ---------------------------- | ------------------------------------------- | -------------------------------- |
| `title`                      | [[String]]`?`                               | Основной текст кнопки            |
| `subtitle`                   | `String?`                                   | Мелкий текст под заголовком      |
| `image`                      | [[UIImage]]`?`                              | Иконка (SF Symbols или кастом)   |
| `imagePlacement`             | `.leading`, `.trailing`, `.top`, `.bottom`  | Где иконка относительно текста   |
| `imagePadding`               | `CGFloat`                                   | Отступ между иконкой и текстом   |
| `baseBackgroundColor`        | [[UIColor]]`?`                              | Цвет фона                        |
| `baseForegroundColor`        | `UIColor?`                                  | Цвет текста и иконки             |
| `cornerStyle`                | `.dynamic`, `.medium`, `.large`, `.capsule` | `.capsule` — модный скруглённый  |
| `buttonSize`                 | `.mini`, `.small`, `.medium`, `.large`      | `.medium` — самый универсальный  |
| `showsActivityIndicator`     | `Bool`                                      | Показывать спиннер вместо текста |
| `activityIndicatorPlacement` | `.leading`, `.trailing`                     | Где спиннер                      |

### 4. Лучшие практики UIButton.Configuration в Swift 2026

- **Всегда используй [[UIAction]]** вместо target-action — это стандарт  
- **Не трогай layer** напрямую — конфигурация сама управляет cornerRadius, shadow и т.д.  
- **Используй configurationUpdateHandler** для динамического изменения вида (highlighted, selected, disabled)  
- **SF Symbols** — с `weight` и `scale` для лучшей читаемости  
- **Доступность** — конфигурация автоматически наследует accessibilityLabel от title/image  
- **[[@MainActor]]** — все изменения кнопок — на главном акторе  
- **[[Swift]] 6 strict concurrency** — UIButton.Configuration полностью безопасен  
- **Документируйте** — пиши комментарий «UIButton.Configuration — кнопка "Сохранить" с иконкой и анимацией»

**Короткий девиз 2026**:
> UIButton.Configuration — это **единый объект**, который заменяет все старые методы setTitle/setImage/layer.  
> В 2026 году это **единственный правильный** способ создавать кнопки в UIKit.  
> Используй `.filled`, `.tinted`, UIAction, configurationUpdateHandler и SF Symbols — забудь старый стиль.
