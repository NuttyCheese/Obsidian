**UIStackView** — это контейнерный класс в [[UIKit]] (доступен с iOS 9, 2015 год), который позволяет **автоматически размещать и выравнивать** набор дочерних представлений (views) в **линейном порядке** — либо горизонтально, либо вертикально.

Это **один из самых часто используемых** и **самых рекомендуемых** компонентов для создания адаптивных, читаемых и поддерживаемых интерфейсов в UIKit-приложениях.

### Основные свойства UIStackView (самые важные в 2026 году)

| Свойство                          | Тип / Значение по умолчанию                          | Что делает / зачем нужно                                      | Самый частый сценарий |
|-----------------------------------|-------------------------------------------------------|---------------------------------------------------------------|-----------------------|
| `axis`                            | `NSLayoutConstraint.Axis` (.horizontal / .vertical)   | Основное направление стека (горизонтально или вертикально)   | Основной выбор при создании |
| `distribution`                    | `UIStackView.Distribution` (.fill, .fillEqually, .fillProportionally, .equalSpacing, .equalCentering) | Как распределять свободное пространство между элементами     | `.fillEqually` — равные размеры, `.fill` — заполнение |
| `alignment`                       | `UIStackView.Alignment` (.fill, .leading, .top, .firstBaseline, .center, .trailing, .bottom, .lastBaseline) | Выравнивание элементов перпендикулярно оси                   | `.center` — самый популярный |
| `spacing`                         | `CGFloat` (0.0)                                       | Фиксированный отступ между элементами                         | 8–16 pt — стандарт для iOS |
| `arrangedSubviews`                | `[UIView]` (только для чтения)                        | Все дочерние views, добавленные через `addArrangedSubview`    | Управление содержимым |
| `isLayoutMarginsRelativeArrangement` | `Bool` (false)                                     | Учитывать layoutMargins при размещении                        | Включать для отступов от краёв |
| `isBaselineRelativeArrangement`   | `Bool` (false)                                        | Выравнивание по базовой линии текста (для UILabel)           | Текст + иконки в ряд |

### Самый популярный и рекомендуемый паттерн 2026 года (UIKit + [[Auto Layout]])

```swift
class ProfileHeaderView: UIView {
    
    private let stackView = UIStackView()
    private let avatarImageView = UIImageView()
    private let nameLabel = UILabel()
    private let statusLabel = UILabel()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupStackView()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        setupStackView()
    }
    
    private func setupStackView() {
        // Основные настройки стека
        stackView.axis = .vertical
        stackView.alignment = .center
        stackView.distribution = .fill
        stackView.spacing = 8
        stackView.isLayoutMarginsRelativeArrangement = true
        stackView.directionalLayoutMargins = NSDirectionalEdgeInsets(top: 16, leading: 16, bottom: 16, trailing: 16)
        stackView.translatesAutoresizingMaskIntoConstraints = false
        addSubview(stackView)
        
        // Настройка дочерних элементов
        avatarImageView.contentMode = .scaleAspectFill
        avatarImageView.layer.cornerRadius = 40
        avatarImageView.clipsToBounds = true
        avatarImageView.backgroundColor = .systemGray5
        
        nameLabel.font = .preferredFont(forTextStyle: .title1)
        nameLabel.textAlignment = .center
        
        statusLabel.font = .preferredFont(forTextStyle: .subheadline)
        statusLabel.textColor = .secondaryLabel
        statusLabel.textAlignment = .center
        
        // Добавляем в стек
        stackView.addArrangedSubview(avatarImageView)
        stackView.addArrangedSubview(nameLabel)
        stackView.addArrangedSubview(statusLabel)
        
        // Constraints для самого stackView
        NSLayoutConstraint.activate([
            stackView.topAnchor.constraint(equalTo: topAnchor),
            stackView.bottomAnchor.constraint(equalTo: bottomAnchor),
            stackView.leadingAnchor.constraint(equalTo: leadingAnchor),
            stackView.trailingAnchor.constraint(equalTo: trailingAnchor),
            
            // Фиксированный размер аватарки
            avatarImageView.widthAnchor.constraint(equalToConstant: 80),
            avatarImageView.heightAnchor.constraint(equalToConstant: 80)
        ])
    }
    
    // Динамическое обновление (например, при смене темы или Dynamic Type)
    override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
        super.traitCollectionDidChange(previousTraitCollection)
        nameLabel.font = .preferredFont(forTextStyle: .title1)
        statusLabel.font = .preferredFont(forTextStyle: .subheadline)
    }
}
```

### Распространённые distribution и alignment (с примерами)

| distribution              | Что делает                                              | Когда использовать в 2026                              |
|---------------------------|----------------------------------------------------------|--------------------------------------------------------|
| `.fill`                   | Заполняет всё доступное пространство (по умолчанию)      | Основной выбор, если размеры задаются вручную          |
| `.fillEqually`            | Все элементы равной ширины/высоты                        | Иконки в ряд, кнопки одинакового размера               |
| `.fillProportionally`     | Пропорционально их intrinsic content size                | Элементы с разным контентом (текст + иконка)           |
| `.equalSpacing`           | Равные отступы между элементами                          | Центрированные иконки с равными промежутками           |
| `.equalCentering`         | Центрирует элементы с равными промежутками до краёв      | Редко, но красиво для симметричных рядов               |

| alignment                 | Когда лучше всего работает                              | Самый частый выбор |
|---------------------------|----------------------------------------------------------|--------------------|
| `.fill`                   | Когда элементы должны растягиваться по перпендикулярной оси | ★★★★★ (самый популярный) |
| `.center`                 | Центрирование по перпендикулярной оси                    | ★★★★☆              |
| `.leading` / `.top`       | Выравнивание по началу оси                               | Текст слева/сверху |
| `.firstBaseline` / `.lastBaseline` | Выравнивание по базовой линии текста             | Ряд UILabel с разными шрифтами |

### Лучшие практики UIStackView в 2026 году

- **Всегда** задавайте `axis`, `alignment`, `distribution` и `spacing` — это определяет 90% поведения  
- **Используйте** `addArrangedSubview` / `removeArrangedSubview` вместо `addSubview` — только arranged subviews участвуют в layout  
- **Для динамического контента** — используйте `arrangedSubviews` для скрытия/показа: `view.isHidden = true` (не удаляйте из стека)  
- **Для вложенных стеков** — комбинируйте горизонтальный и вертикальный — это основа большинства современных экранов  
- **Для SwiftUI** — используйте `HStack` / `VStack` — UIStackView нужен только в UIKit или смешанных проектах  
- **Для производительности** — избегайте слишком глубокого nesting (больше 5–6 уровней стеков) — лучше использовать Auto Layout constraints  
- **Документируйте** — пишите комментарий:

```swift
/// Вертикальный стек для профиля: аватарка + имя + статус
private let profileStack = UIStackView()
```

**Короткий итог 2026**:
> `UIStackView` — это **линейный контейнер** для автоматического размещения и выравнивания дочерних views.  
> В 2026 году:  
> - ключевые свойства — `axis`, `distribution`, `alignment`, `spacing`  
> - самый популярный — `.vertical`, `.fill`, `.center`, spacing 8–16 pt  
> - идеален для форм, карточек, рядов кнопок, профилей, настроек  
> - это **основа** большинства экранов в UIKit-приложениях  
