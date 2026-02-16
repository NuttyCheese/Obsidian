**Auto Layout** — это система автоматического размещения и адаптации пользовательского интерфейса в **[[UIKit]]** ([[iOS]], iPadOS, macOS Catalyst) и частично в **[[AppKit]]** (macOS).

По состоянию на 2026 год Auto Layout остаётся **основным и самым надёжным** способом создания адаптивных интерфейсов в UIKit-приложениях, особенно для поддержки всех размеров экранов, ориентаций, динамического типа текста, тёмной/светлой темы и локализации.

### Основные принципы Auto Layout (актуальные в 2026)

Auto Layout работает на основе **ограничений** (constraints), которые описывают отношения между элементами UI и их супервидами или константами.

Ключевые правила:

1. Каждый вид должен иметь **достаточно ограничений**, чтобы однозначно определить его **позицию** (x, y) и **размер** (width, height)  
2. Система решает уравнения в runtime → если ограничений недостаточно или они конфликтуют → **unsatisfiable constraints** или **ambiguous layout**  
3. Приоритет ограничений (от 1 до 1000) позволяет разрешать конфликты  
4. **Intrinsic Content Size** — встроенный размер ([[UILabel]], [[UIImageView]] и т.д.) помогает системе автоматически определять width/height

### Основные способы создания Auto Layout в 2026 году (от самого рекомендуемого к legacy)

| Способ                                        | Год появления | Уровень читаемости | Скорость написания | Поддержка Swift 6+ concurrency | Рекомендация 2026        | Пример использования      |
| --------------------------------------------- | ------------- | ------------------ | ------------------ | ------------------------------ | ------------------------ | ------------------------- |
| **NSLayoutConstraint.activate** (программный) | 2014          | ★★★★★              | ★★★★★              | ★★★★★                          | Основной выбор           | Всё новое и сложное       |
| **[[Interface Builder]] / Storyboard**        | 2011          | ★★★★☆              | ★★★★☆              | ★★★★☆                          | Для простых экранов      | Прототипы, быстрые экраны |
| **[[NSLayoutAnchor]]** (синтаксический сахар) | 2015          | ★★★★★              | ★★★★★              | ★★★★★                          | Самый читаемый           | Программный UI            |
| **VFL** (Visual Format Language)              | 2011          | ★★☆☆☆              | ★★★☆☆              | ★★★★☆                          | Legacy, не рекомендуется | Старый код                |
| **Masonry / SnapKit** (第三方)                   | 2013–2015     | ★★★★☆              | ★★★★★              | ★★★★☆                          | Только в legacy          | Старые проекты            |

### Самые популярные и рекомендуемые паттерны Auto Layout в 2026

#### Паттерн 1 — Самый современный и читаемый (NSLayoutAnchor + activate)

```swift
class ProfileHeaderView: UIView {
    
    let avatarImageView = UIImageView()
    let nameLabel = UILabel()
    let bioLabel = UILabel()
    
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
            // Avatar — слева сверху
            avatarImageView.topAnchor.constraint(equalTo: topAnchor, constant: 16),
            avatarImageView.leadingAnchor.constraint(equalTo: leadingAnchor, constant: 16),
            avatarImageView.widthAnchor.constraint(equalToConstant: 80),
            avatarImageView.heightAnchor.constraint(equalToConstant: 80),
            
            // Name — справа от аватара
            nameLabel.leadingAnchor.constraint(equalTo: avatarImageView.trailingAnchor, constant: 16),
            nameLabel.topAnchor.constraint(equalTo: avatarImageView.topAnchor),
            nameLabel.trailingAnchor.constraint(equalTo: trailingAnchor, constant: -16),
            
            // Bio — под именем
            bioLabel.leadingAnchor.constraint(equalTo: nameLabel.leadingAnchor),
            bioLabel.topAnchor.constraint(equalTo: nameLabel.bottomAnchor, constant: 8),
            bioLabel.trailingAnchor.constraint(equalTo: trailingAnchor, constant: -16),
            bioLabel.bottomAnchor.constraint(equalTo: bottomAnchor, constant: -16)
        ])
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

#### Паттерн 2 — Использование layout guides и safe area

```swift
NSLayoutConstraint.activate([
    button.leadingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.leadingAnchor, constant: 20),
    button.trailingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.trailingAnchor, constant: -20),
    button.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor, constant: -20),
    button.heightAnchor.constraint(equalToConstant: 50)
])
```

#### Паттерн 3 — Динамическая высота (intrinsic size + content hugging/compression)

```swift
bioLabel.setContentHuggingPriority(.defaultHigh, for: .horizontal)
bioLabel.setContentCompressionResistancePriority(.defaultLow, for: .horizontal)
```

### Лучшие практики Auto Layout в Swift 2026

- **translatesAutoresizingMaskIntoConstraints = false** — всегда для programmatic UI  
- **NSLayoutConstraint.activate** — группируй все ограничения в одном месте  
- **Safe Area** — используй `safeAreaLayoutGuide` вместо `layoutMarginsGuide`  
- **Priority** — используй 750–999 для основных, 250–749 для fallback  
- **Content Hugging / Compression Resistance** — обязательно для динамического контента  
- **@MainActor** — все обновления Auto Layout должны быть на главном потоке  
- **Swift 6 strict concurrency** — избегай захвата self в замыканиях без [weak self]  
- **Тестирование** — используй **UI-тесты** + **snapshot-тесты** для проверки layout на разных размерах/темах  
- **Не используй** VFL в новом коде — слишком много ошибок и плохая читаемость

**Короткий девиз 2026**:
> «Auto Layout в 2026 году — это когда ты пишешь адаптивный UI один раз и он работает на всех устройствах, ориентациях, темах и локализациях.  
> Самый современный стиль — NSLayoutAnchor + programmatic constraints + safe area.»
