**UITraitCollection** — это объект в [[UIKit]], который содержит набор **черт окружения** (traits) текущего устройства или экрана. Он описывает контекст, в котором отображается интерфейс приложения:

- размер экрана и классы размеров (size classes),
- светлая/тёмная тема,
- размер текста (Dynamic Type),
- направление письма (LTR/RTL),
- масштаб дисплея,
- ориентация,
- и другие характеристики.

`UITraitCollection` позволяет адаптировать интерфейс под разные устройства (iPhone / iPad), режимы (split view, внешний дисплей), темы и настройки доступности.

### Основные черты (traits) в UITraitCollection (2026 год)

| Черта (свойство)                            | Тип значения                                                      | Когда меняется / зачем важно                         | Самый частый сценарий адаптации           |
| ------------------------------------------- | ----------------------------------------------------------------- | ---------------------------------------------------- | ----------------------------------------- |
| `horizontalSizeClass` / `verticalSizeClass` | `UIUserInterfaceSizeClass` (.compact / .regular)                  | iPhone → iPad, Split View, Slide Over, Stage Manager | Разные layout: одна колонка / две колонки |
| `userInterfaceStyle`                        | `UIUserInterfaceStyle` (.light / .dark / .unspecified)            | Смена светлой/тёмной темы                            | Цвета, изображения, blur-эффекты          |
| `preferredContentSizeCategory`              | `UIContentSizeCategory` (.small ... .accessibilityXXXL)           | Пользователь меняет размер текста в настройках       | Динамические шрифты (Dynamic Type)        |
| `layoutDirection`                           | `UITraitEnvironmentLayoutDirection` (.leftToRight / .rightToLeft) | Поддержка арабского, иврита и других RTL-языков      | Зеркальное расположение элементов         |
| `displayScale`                              | [[CGFloat]] (1x, 2x, 3x)                                          | Разные устройства (Retina, Super Retina XDR)         | Выбор @2x/@3x изображений                 |
| `activeAppearance` (iOS 17+)                | `UIUserInterfaceActiveAppearance` (.active / .inactive)           | Приложение в фоне / на экране блокировки             | Приостановка анимаций                     |
| `legibilityWeight`                          | `UILegibilityWeight` (.regular / .bold)                           | Пользователь включает жирный текст                   | Жирность шрифтов                          |
| `accessibilityContrast` (iOS 13+)           | `UIAccessibilityContrast` (.normal / .high)                       | Высокий контраст в настройках доступности            | Увеличение контраста цветов               |

### Самые важные методы и свойства (2026)

| Метод / Свойство                                  | Что возвращает / делает                                      | Самый частый сценарий |
|---------------------------------------------------|---------------------------------------------------------------|-----------------------|
| `traitCollectionDidChange(_:)`                    | Вызывается при изменении любой черты                         | Адаптация layout, цветов, шрифтов |
| `hasDifferentColorAppearance(comparedTo:)`        | `Bool` — изменилась ли тёмная/светлая тема                   | Переключение цветов/изображений |
| `UITraitCollection.current`                       | Текущая коллекция черт (в контроллере или вью)               | Быстрый доступ к traitCollection |
| `UITraitCollection(userInterfaceStyle:)`          | Создаёт коллекцию только с заданной темой                    | Тестирование тёмной темы |
| `UITraitCollection(preferredContentSizeCategory:)` | Создаёт коллекцию с заданным размером текста                 | Тестирование Dynamic Type |
| `UITraitCollection(traitsFrom:)`                  | Комбинирует несколько коллекций черт                         | Сложные сценарии адаптации |

### Самый популярный и рекомендуемый паттерн 2026 года  
(Адаптация интерфейса в [[UIViewController]] / [[UIView]])

```swift
class AdaptiveProfileViewController: UIViewController {
    
    private let avatarImageView = UIImageView()
    private let nameLabel = UILabel()
    private let bioTextView = UITextView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Начальная настройка под текущие черты
        updateInterface(for: traitCollection)
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
            updateTextSizes()
        }
    }
    
    private func updateLayout(for traitCollection: UITraitCollection) {
        if traitCollection.horizontalSizeClass == .regular {
            // iPad / landscape → две колонки
            stackView.axis = .horizontal
            avatarImageView.widthAnchor.constraint(equalToConstant: 120).isActive = true
        } else {
            // iPhone → одна колонка
            stackView.axis = .vertical
            avatarImageView.widthAnchor.constraint(equalToConstant: 100).isActive = true
        }
    }
    
    private func updateColors() {
        view.backgroundColor = traitCollection.userInterfaceStyle == .dark ? .systemBackground : .secondarySystemBackground
        nameLabel.textColor = .label
        bioTextView.textColor = .secondaryLabel
    }
    
    private func updateTextSizes() {
        nameLabel.font = .preferredFont(forTextStyle: .title1)
        nameLabel.adjustsFontForContentSizeCategory = true
        
        bioTextView.font = .preferredFont(forTextStyle: .body)
        bioTextView.adjustsFontForContentSizeCategory = true
    }
}
```

### Лучшие практики UITraitCollection в 2026 году

- **Всегда** вызывайте `super.traitCollectionDidChange(previousTraitCollection)`  
- **Сравнивайте** с `previousTraitCollection` — чтобы не делать лишнюю работу при каждом вызове  
- **Для Dynamic Type** — используйте `preferredFont(forTextStyle:)` и `adjustsFontForContentSizeCategory = true`  
- **Для тёмной темы** — используйте системные цвета (`.label`, `.systemBackground`) или `traitCollection.userInterfaceStyle`  
- **Для size class** — проверяйте `horizontalSizeClass` и `verticalSizeClass` отдельно  
- **Для SwiftUI** — используйте `@Environment(\.horizontalSizeClass)`, `@Environment(\.colorScheme)`, `@Environment(\.dynamicTypeSize)` — `UITraitCollection` нужен только в UIKit  
- **Документируйте** — пишите комментарий:

```swift
override func traitCollectionDidChange(_ previous: UITraitCollection?) {
    super.traitCollectionDidChange(previous)
    
    // Адаптируем layout и цвета под новые черты окружения
    updateLayoutAndAppearance()
}
```

**Короткий итог 2026**:
> `UITraitCollection` — объект, содержащий **все черты окружения** (size class, тёмная тема, Dynamic Type, RTL и т.д.).  
> В 2026 году:  
> - основной метод реакции — `traitCollectionDidChange(_:)`  
> - сравнивайте с `previousTraitCollection`, чтобы оптимизировать  
> - критично для адаптивного UI на iPhone / iPad / внешних дисплеях  
> - это **основа** responsive и доступного дизайна в UIKit-приложениях  
