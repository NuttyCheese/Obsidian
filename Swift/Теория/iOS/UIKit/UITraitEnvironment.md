**UITraitEnvironment** — это протокол в [[UIKit]], который позволяет объектам (в первую очередь [[UIViewController]] и [[UIView]]) **реагировать на изменения черт окружения** (traits) — таких как размер экрана, ориентация, тёмная/светлая тема, размер динамического текста, scale экрана и т.д.

Это **один из самых важных** механизмов адаптивного интерфейса в [[iOS]], начиная с iOS 8 и особенно актуальный в 2026 году с поддержкой iPadOS, Stage Manager, внешних дисплеев и Dynamic Island.

### Основная идея и жизненный цикл

Протокол `UITraitEnvironment` добавляет два ключевых элемента:

1. **Свойство `traitCollection`**  
   ```swift
   var traitCollection: UITraitCollection { get }
   ```
   Текущий набор черт (size class, userInterfaceStyle, preferredContentSizeCategory и т.д.)

2. **Метод `traitCollectionDidChange(_:)`**  
   ```swift
   func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?)
   ```
   Вызывается системой каждый раз, когда черты окружения изменились.

### Кто реализует UITraitEnvironment

| Класс / Тип                  | Реализует протокол? | Самый частый сценарий использования traitCollectionDidChange       |
| ---------------------------- | ------------------- | ------------------------------------------------------------------ |
| [[UIViewController]]         | Да                  | Адаптация layout, скрытие/показ элементов в разных size class      |
| [[UIView]]                   | Да                  | Изменение шрифтов, отступов, цвета при смене темы или Dynamic Type |
| [[UIPresentationController]] | Да                  | Кастомные модальные контроллеры (bottom sheet и т.д.)              |
| [[UIWindow]]                 | Да                  | Редко, но полезно для глобальных изменений                         |
| [[UIScreen]]                 | Нет (но связан)     | Через `traitCollection` окна                                       |

### Самые важные черты (traits), на которые реагируют в 2026

| UITraitCollection свойство                | Тип значения                                   | Когда меняется                                      | Самый частый кейс адаптации |
|-------------------------------------------|------------------------------------------------|-----------------------------------------------------|-----------------------------|
| `horizontalSizeClass` / `verticalSizeClass` | `UIUserInterfaceSizeClass` (.compact / .regular) | iPhone → iPad, Split View, Slide Over               | Разные layout для телефона и планшета |
| `userInterfaceStyle`                      | `UIUserInterfaceStyle` (.light / .dark / .unspecified) | Смена светлой/тёмной темы                           | Цвета, изображения, blur-эффекты |
| `preferredContentSizeCategory`            | `UIContentSizeCategory` (.small ... .accessibilityXXXL) | Пользователь меняет размер текста в настройках      | Динамические шрифты (Dynamic Type) |
| `layoutDirection`                         | `UITraitEnvironmentLayoutDirection` (.leftToRight / .rightToLeft) | Поддержка RTL-языков (арабский, иврит)              | Зеркальное расположение элементов |
| `displayScale`                            | `CGFloat` (1x, 2x, 3x)                         | Подключение внешнего дисплея                        | Качество изображений |
| `activeAppearance` (iOS 17+)              | `UIUserInterfaceActiveAppearance` (.active / .inactive) | Приложение в фоне / на экране блокировки            | Приостановка анимаций |
| `legibilityWeight`                        | `UILegibilityWeight` (.bold / .regular)        | Пользователь включает жирный текст                  | Жирность шрифтов |

### Самый популярный и рекомендуемый паттерн 2026 года

#### 1. Адаптация layout в UIViewController

```swift
class AdaptiveViewController: UIViewController {
    
    private let stackView = UIStackView()
    private let imageView = UIImageView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        stackView.axis = .vertical
        stackView.spacing = 16
        
        // Начальная настройка
        updateLayout(for: traitCollection)
        
        view.addSubview(stackView)
        // ... constraints
    }
    
    override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
        super.traitCollectionDidChange(previousTraitCollection)
        
        // Проверяем, что именно изменилось
        if traitCollection.hasDifferentColorAppearance(comparedTo: previousTraitCollection) {
            updateColors()
        }
        
        if traitCollection.horizontalSizeClass != previousTraitCollection?.horizontalSizeClass ||
           traitCollection.verticalSizeClass != previousTraitCollection?.verticalSizeClass {
            updateLayout(for: traitCollection)
        }
        
        if traitCollection.preferredContentSizeCategory != previousTraitCollection?.preferredContentSizeCategory {
            updateFonts()
        }
    }
    
    private func updateLayout(for traitCollection: UITraitCollection) {
        if traitCollection.horizontalSizeClass == .regular {
            stackView.axis = .horizontal
            imageView.contentMode = .scaleAspectFit
        } else {
            stackView.axis = .vertical
            imageView.contentMode = .scaleAspectFill
        }
    }
    
    private func updateColors() {
        view.backgroundColor = traitCollection.userInterfaceStyle == .dark ? .systemBackground : .secondarySystemBackground
    }
    
    private func updateFonts() {
        // Используем preferredFont(forTextStyle:) — автоматически адаптируется
        titleLabel.font = .preferredFont(forTextStyle: .title1)
    }
}
```

#### 2. Адаптация в UIView (часто забывают!)

```swift
class AdaptiveCardView: UIView {
    
    private let titleLabel = UILabel()
    
    override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
        super.traitCollectionDidChange(previousTraitCollection)
        
        // Реагируем на Dynamic Type
        titleLabel.font = .preferredFont(forTextStyle: .headline)
        titleLabel.adjustsFontForContentSizeCategory = true
        
        // Реагируем на тёмную тему
        backgroundColor = traitCollection.userInterfaceStyle == .dark ? .systemGray6 : .systemGray5
    }
}
```

### Лучшие практики UITraitEnvironment в 2026

- **Всегда** вызывайте `super.traitCollectionDidChange(previousTraitCollection)` — иначе сломаете поведение суперкласса  
- **Проверяйте** именно то, что изменилось — сравнивайте с `previousTraitCollection`  
- **Для Dynamic Type** — используйте `preferredFont(forTextStyle:)` и `adjustsFontForContentSizeCategory = true`  
- **Для тёмной темы** — используйте `traitCollection.userInterfaceStyle` или системные цвета (.systemBackground, .label)  
- **Для size class** — проверяйте `horizontalSizeClass` и `verticalSizeClass` отдельно  
- **В SwiftUI** — используйте `@Environment(\.horizontalSizeClass)`, `@Environment(\.colorScheme)` — `UITraitEnvironment` нужен только в UIKit  
- **Документируйте** — пишите комментарий:

```swift
override func traitCollectionDidChange(_ previous: UITraitCollection?) {
    super.traitCollectionDidChange(previous)
    
    // Адаптируем layout под size class и тему
    updateLayoutAndAppearance()
}
```

**Короткий итог 2026**:
> `UITraitEnvironment` — протокол для реакции на изменения черт окружения (size class, тёмная тема, Dynamic Type, RTL и т.д.).  
> В 2026 году:  
> - основной метод — `traitCollectionDidChange(_:)`  
> - сравнивайте с `previousTraitCollection`, чтобы не делать лишнюю работу  
> - критично для адаптивного UI на iPhone / iPad / внешних дисплеях  
> - это **основа** responsive-дизайна в [[UIKit]]-приложениях  
