**NSLayoutAnchor** — это современный, типобезопасный и самый рекомендуемый способ создания **Auto Layout constraints** в [[UIKit]] начиная с [[iOS]] 9 (2015 год). К 2026 году это **де-факто стандарт** для программного Auto Layout в [[Swift]].

Он полностью заменил старые способы (VFL, `NSLayoutConstraint` с константами вручную) благодаря читаемости, безопасности типов и удобству цепочек.

### Основные классы NSLayoutAnchor

| Класс                          | Что привязывает                          | Самые частые методы                          | Пример использования |
|--------------------------------|------------------------------------------|----------------------------------------------|----------------------|
| **NSLayoutXAxisAnchor**        | Горизонтальные позиции (leading, trailing, left, right, centerX) | `constraint(equalTo:)`, `constraint(equalToConstant:)` | leadingAnchor, trailingAnchor, centerXAnchor |
| **NSLayoutYAxisAnchor**        | Вертикальные позиции (top, bottom, centerY, firstBaseline, lastBaseline) | `constraint(equalTo:)`, `constraint(greaterThanOrEqualTo:)` | topAnchor, bottomAnchor, centerYAnchor |
| **NSLayoutDimension**          | Размеры (width, height)                  | `constraint(equalTo:)`, `constraint(equalToConstant:)`, `constraint(equalToMultiplier:)` | widthAnchor, heightAnchor |
| **NSLayoutAnchor** (базовый)   | Общий предок для всех выше               | —                                            | — |

### Самый популярный и рекомендуемый паттерн 2026 года

```swift
class ProfileHeaderView: UIView {
    
    private let avatarImageView = UIImageView()
    private let nameLabel = UILabel()
    private let bioLabel = UILabel()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupViews()
        setupConstraints()
    }
    
    private func setupViews() {
        avatarImageView.translatesAutoresizingMaskIntoConstraints = false
        nameLabel.translatesAutoresizingMaskIntoConstraints = false
        bioLabel.translatesAutoresizingMaskIntoConstraints = false
        
        addSubview(avatarImageView)
        addSubview(nameLabel)
        addSubview(bioLabel)
        
        avatarImageView.contentMode = .scaleAspectFill
        avatarImageView.layer.cornerRadius = 40
        avatarImageView.clipsToBounds = true
        
        nameLabel.font = .preferredFont(forTextStyle: .headline)
        bioLabel.font = .preferredFont(forTextStyle: .subheadline)
        bioLabel.numberOfLines = 0
    }
    
    private func setupConstraints() {
        NSLayoutConstraint.activate([
            // Аватарка слева сверху
            avatarImageView.topAnchor.constraint(equalTo: safeAreaLayoutGuide.topAnchor, constant: 16),
            avatarImageView.leadingAnchor.constraint(equalTo: leadingAnchor, constant: 16),
            avatarImageView.widthAnchor.constraint(equalToConstant: 80),
            avatarImageView.heightAnchor.constraint(equalToConstant: 80),
            
            // Имя — справа от аватарки, выровнено по верху
            nameLabel.leadingAnchor.constraint(equalTo: avatarImageView.trailingAnchor, constant: 16),
            nameLabel.topAnchor.constraint(equalTo: avatarImageView.topAnchor),
            nameLabel.trailingAnchor.constraint(equalTo: trailingAnchor, constant: -16),
            
            // Био — под именем, растягивается по ширине
            bioLabel.leadingAnchor.constraint(equalTo: nameLabel.leadingAnchor),
            bioLabel.topAnchor.constraint(equalTo: nameLabel.bottomAnchor, constant: 8),
            bioLabel.trailingAnchor.constraint(equalTo: trailingAnchor, constant: -16),
            bioLabel.bottomAnchor.constraint(equalTo: safeAreaLayoutGuide.bottomAnchor, constant: -16)
        ])
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

### Самые полезные методы и модификаторы NSLayoutAnchor

```swift
// Основные отношения
leadingAnchor.constraint(equalTo: other.leadingAnchor, constant: 16)
topAnchor.constraint(greaterThanOrEqualTo: other.topAnchor)
centerXAnchor.constraint(equalTo: superview.centerXAnchor)

// Размеры
widthAnchor.constraint(equalToConstant: 200)
heightAnchor.constraint(equalTo: widthAnchor, multiplier: 1.5)  // пропорция 2:3

// Приоритет и идентификатор (очень полезно для отладки)
constraint.priority = .defaultHigh
constraint.identifier = "avatar-top-constraint"

// Активация сразу нескольких
NSLayoutConstraint.activate([
    view.topAnchor.constraint(equalTo: superview.topAnchor),
    view.leadingAnchor.constraint(equalTo: superview.leadingAnchor)
])
```

### Лучшие практики NSLayoutAnchor в Swift 2026

- **translatesAutoresizingMaskIntoConstraints = false** — обязательно для всех programmatic вью  
- **NSLayoutConstraint.activate([...])** — активируй сразу массивом (одна операция)  
- **safeAreaLayoutGuide / readableContentGuide** — вместо прямого topAnchor к superview  
- **identifier** — задавай для всех важных констрейнтов (видно в отладчике и логах)  
- **priority** — используй .required (1000), .high (750), .low (250) для гибкости  
- **@MainActor** — все операции с constraints — на главном акторе  
- **Swift 6 strict concurrency** — NSLayoutAnchor / NSLayoutConstraint полностью Sendable-safe  
- **Тестирование** — используй snapshot-тесты (iOSSnapshotTestCase) для проверки layout  
- **Документируйте** — пиши комментарий «NSLayoutAnchor — программный Auto Layout с safe area»

**Короткий девиз 2026**:
> «NSLayoutAnchor — это когда ты пишешь Auto Layout программно так, чтобы код читался как английское предложение:  
> avatarImageView.topAnchor.constraint(equalTo: safeAreaLayoutGuide.topAnchor, constant: 16)  
> В 2026 году это **единственный современный** способ создавать constraints в UIKit.  
> Забудь VFL и старые NSLayoutConstraint.init — только anchors + activate.»
