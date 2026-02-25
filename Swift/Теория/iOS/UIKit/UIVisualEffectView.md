**UIVisualEffectView** — это специальный подкласс [[UIView]], который позволяет применять **визуальные эффекты размытия** (blur) и/или **вибрации** (vibrancy) к содержимому, расположенному под ним.

Это один из самых популярных и часто используемых компонентов для создания современного, «стеклянного» (glassmorphism) или полупрозрачного интерфейса в iOS-приложениях.

### Основные возможности UIVisualEffectView (актуально на 2026 год)

| Свойство / Эффект                     | Тип эффекта                                      | Когда выглядит лучше всего                          | Самый частый сценарий |
|---------------------------------------|--------------------------------------------------|-----------------------------------------------------|-----------------------|
| `UIBlurEffect(style: .systemMaterial)` | Лёгкое/среднее/сильное размытие + адаптация под свет/тёмную тему | Почти всегда — современный стандарт iOS 18+         | Фон панелей, боковых меню, модальных окон |
| `.systemUltraThinMaterial`            | Очень лёгкое размытие                            | Светлые интерфейсы, минимализм                      | Карточки, плавающие кнопки |
| `.systemThinMaterial`                 | Лёгкое размытие                                  | Баланс между читаемостью и эффектом                 | Навигационные панели, тулбары |
| `.systemMaterial`                     | Среднее размытие (самый популярный)              | Универсальный выбор для большинства случаев         | Bottom sheets, sidebars, search bar background |
| `.systemThickMaterial`                | Сильное размытие                                 | Тёмные интерфейсы, когда нужен сильный контраст     | Контрастные оверлеи, полноэкранные модалки |
| `.systemChromeMaterial`               | Размытие как у системных элементов (Control Center) | Когда нужно максимально соответствовать системному стилю | Кастомные Control Center-подобные панели |
| `UIVibrancyEffect(blurEffect:)`       | Усиление контраста текста/иконок на размытом фоне | Текст и иконки поверх blur                          | Надписи на размытом фоне (часто с .label) |

### Самый популярный и рекомендуемый паттерн 2026 года

```swift
class ModernCardView: UIView {
    
    private let blurView = UIVisualEffectView()
    private let vibrancyContent = UIVisualEffectView()
    private let contentStack = UIStackView()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupBlurAndVibrancy()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        setupBlurAndVibrancy()
    }
    
    private func setupBlurAndVibrancy() {
        // 1. Основной эффект размытия (самый популярный в 2026)
        blurView.effect = UIBlurEffect(style: .systemMaterial)
        blurView.translatesAutoresizingMaskIntoConstraints = false
        addSubview(blurView)
        
        // 2. Контейнер для vibrancy (чтобы текст и иконки выглядели ярче)
        vibrancyContent.effect = UIVibrancyEffect(blurEffect: UIBlurEffect(style: .systemMaterial))
        vibrancyContent.translatesAutoresizingMaskIntoConstraints = false
        blurView.contentView.addSubview(vibrancyContent)
        
        // 3. Контент (текст, иконки и т.д.)
        contentStack.axis = .vertical
        contentStack.spacing = 12
        contentStack.translatesAutoresizingMaskIntoConstraints = false
        vibrancyContent.contentView.addSubview(contentStack)
        
        // Добавляем пример контента
        let titleLabel = UILabel()
        titleLabel.text = "Современная карточка"
        titleLabel.font = .systemFont(ofSize: 20, weight: .bold)
        
        let subtitleLabel = UILabel()
        subtitleLabel.text = "Размытие + vibrancy"
        subtitleLabel.font = .systemFont(ofSize: 16)
        subtitleLabel.textColor = .secondaryLabel
        
        contentStack.addArrangedSubview(titleLabel)
        contentStack.addArrangedSubview(subtitleLabel)
        
        // Layout
        NSLayoutConstraint.activate([
            blurView.topAnchor.constraint(equalTo: topAnchor),
            blurView.bottomAnchor.constraint(equalTo: bottomAnchor),
            blurView.leadingAnchor.constraint(equalTo: leadingAnchor),
            blurView.trailingAnchor.constraint(equalTo: trailingAnchor),
            
            vibrancyContent.topAnchor.constraint(equalTo: blurView.contentView.topAnchor),
            vibrancyContent.bottomAnchor.constraint(equalTo: blurView.contentView.bottomAnchor),
            vibrancyContent.leadingAnchor.constraint(equalTo: blurView.contentView.leadingAnchor),
            vibrancyContent.trailingAnchor.constraint(equalTo: blurView.contentView.trailingAnchor),
            
            contentStack.topAnchor.constraint(equalTo: vibrancyContent.contentView.topAnchor, constant: 20),
            contentStack.bottomAnchor.constraint(equalTo: vibrancyContent.contentView.bottomAnchor, constant: -20),
            contentStack.leadingAnchor.constraint(equalTo: vibrancyContent.contentView.leadingAnchor, constant: 20),
            contentStack.trailingAnchor.constraint(equalTo: vibrancyContent.contentView.trailingAnchor, constant: -20)
        ])
        
        // Важно для красивого скругления
        layer.cornerRadius = 16
        clipsToBounds = true
    }
}
```

### Лучшие практики UIVisualEffectView в 2026 году

- **Используйте** `.systemMaterial` / `.systemThinMaterial` / `.systemUltraThinMaterial` — это адаптивные стили под свет/тёмную тему и Dynamic Type  
- **Для текста и иконок** — всегда оборачивайте в `UIVibrancyEffect` — иначе на сильном размытии они теряются  
- **Не забывайте** `clipsToBounds = true` + `layer.cornerRadius` — размытие красиво скругляется только с этим  
- **Для производительности** — избегайте наложения множества blur-эффектов друг на друга (особенно в скролле)  
- **Для SwiftUI** — используйте `.background(.ultraThinMaterial)` / `.background(.regularMaterial)` — UIVisualEffectView нужен только в UIKit или смешанных проектах  
- **Для адаптивности** — слушайте `traitCollectionDidChange` и обновляйте `effect` при смене темы  
- **Документируйте** — пишите комментарий:

```swift
/// UIVisualEffectView с systemMaterial + vibrancy для текста и иконок
private let blurView = UIVisualEffectView(effect: UIBlurEffect(style: .systemMaterial))
```

**Короткий итог 2026**:
> `UIVisualEffectView` — это системный компонент для **размытия** и **вибрации** фона.  
> В 2026 году:  
> - самый популярный эффект — `.systemMaterial` (адаптивный под свет/тёмную тему)  
> - для текста/иконок — всегда используйте `UIVibrancyEffect`  
> - обязательно `clipsToBounds + cornerRadius` для красивого скругления  
> - в SwiftUI — `.background(.regularMaterial)` / `.ultraThinMaterial`  
> - это **must-have** для современного, «стеклянного» дизайна в iOS-приложениях  
