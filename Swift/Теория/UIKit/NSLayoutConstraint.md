**NSLayoutConstraint** — это основной объект в **UIKit [[Auto Layout]]**, который описывает **одно математическое правило** расположения или размера элемента интерфейса.

Каждое ограничение — это уравнение вида:

**item1.attribute1 = multiplier × item2.attribute2 + constant**

Примеры:
- ведущий край кнопки = ведущий край родителя + 16  
- ширина картинки = высота картинки × 1.618 (золотое сечение)  
- центр метки = центр родителя

### Почему NSLayoutConstraint всё ещё важен в 2026 году

- Это **низкоуровневый фундамент** всего Auto Layout (даже [[NSLayoutAnchor]] и SnapKit под капотом создают именно [[NSLayoutConstraint]])
- Позволяет **динамически менять** констрейнты в runtime (constant, priority, isActive)
- Необходим для **сложных случаев**, где Anchor [[API]] недостаточно гибок
- Используется в **legacy-коде**, в **Storyboard/[[XIB]]** и в **тестах**

### Сравнение способов создания констрейнтов (2026 реальность)

| Способ создания                          | Удобство чтения | Гибкость | Скорость написания | Рекомендация 2026 |
|------------------------------------------|------------------|----------|---------------------|-------------------|
| **NSLayoutAnchor** (view.topAnchor…)     | ★★★★★           | ★★★★☆   | ★★★★☆              | Основной выбор для programmatic UI |
| **NSLayoutConstraint(item:attribute…)**  | ★★☆☆☆           | ★★★★★   | ★★☆☆☆              | Только если нужен очень сложный multiplier/relatedBy |
| **NSLayoutConstraint.activate([…])**     | ★★★★☆           | ★★★★☆   | ★★★★☆              | Когда много констрейнтов сразу |
| **SnapKit**                              | ★★★★★           | ★★★★☆   | ★★★★★              | Если команда любит читаемый код |
| **Storyboard/XIB**                       | ★★★☆☆           | ★★☆☆☆   | ★★★★☆ (drag&drop)  | Только для быстрого прототипа |

### Самый частый и правильный паттерн 2026 (NSLayoutAnchor + activate)

```swift
class ProfileHeaderView: UIView {
    
    private let avatar = UIImageView()
    private let nameLabel = UILabel()
    private let bioLabel = UILabel()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupSubviews()
        setupConstraints()
    }
    
    private func setupSubviews() {
        [avatar, nameLabel, bioLabel].forEach {
            $0.translatesAutoresizingMaskIntoConstraints = false
            addSubview($0)
        }
        
        avatar.contentMode = .scaleAspectFill
        avatar.layer.cornerRadius = 40
        avatar.clipsToBounds = true
        
        nameLabel.font = .preferredFont(forTextStyle: .headline)
        bioLabel.font = .preferredFont(forTextStyle: .subheadline)
        bioLabel.numberOfLines = 0
    }
    
    private func setupConstraints() {
        NSLayoutConstraint.activate([
            // Аватарка
            avatar.topAnchor.constraint(equalTo: safeAreaLayoutGuide.topAnchor, constant: 16),
            avatar.centerXAnchor.constraint(equalTo: centerXAnchor),
            avatar.widthAnchor.constraint(equalToConstant: 80),
            avatar.heightAnchor.constraint(equalTo: avatar.widthAnchor), // квадрат
            
            // Имя под аватаркой
            nameLabel.topAnchor.constraint(equalTo: avatar.bottomAnchor, constant: 12),
            nameLabel.leadingAnchor.constraint(equalTo: leadingAnchor, constant: 16),
            nameLabel.trailingAnchor.constraint(equalTo: trailingAnchor, constant: -16),
            
            // Био под именем
            bioLabel.topAnchor.constraint(equalTo: nameLabel.bottomAnchor, constant: 8),
            bioLabel.leadingAnchor.constraint(equalTo: nameLabel.leadingAnchor),
            bioLabel.trailingAnchor.constraint(equalTo: nameLabel.trailingAnchor),
            bioLabel.bottomAnchor.constraint(equalTo: safeAreaLayoutGuide.bottomAnchor, constant: -16)
        ])
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

### Динамическое управление констрейнтами (самое важное)

```swift
// Создаём и сохраняем
private var heightConstraint: NSLayoutConstraint!

override func setupConstraints() {
    heightConstraint = someView.heightAnchor.constraint(equalToConstant: 100)
    heightConstraint.isActive = true
}

// Позже можно менять
func updateHeight(to newHeight: CGFloat) {
    heightConstraint.constant = newHeight
    // НЕ нужно вызывать layoutIfNeeded() — система сама обновит на следующем кадре
    // Если нужно СЕЙЧАС → layoutIfNeeded()
}

// Изменение priority (очень полезно для адаптивного дизайна)
func makeHeightOptional() {
    heightConstraint.priority = .defaultHigh  // 750 вместо 1000
}
```

### Лучшие практики NSLayoutConstraint в 2026

- **translatesAutoresizingMaskIntoConstraints = false** — всегда для programmatic вью  
- **NSLayoutConstraint.activate([...])** — активируй сразу массивом (быстрее и чище)  
- **safeAreaLayoutGuide / readableContentGuide** — вместо прямой привязки к superview  
- **identifier** — задавай для отладки

```swift
let constraint = view.widthAnchor.constraint(equalToConstant: 200)
constraint.identifier = "profile-avatar-width"
constraint.isActive = true
```

- **priority** — используй .required (1000), .high (750), .low (250)  
- **constant** — положительный = внутрь, отрицательный = наружу  
- **multiplier** — для пропорций (width = height × 1.618)  
- **isActive = false** — для отключения констрейнта без удаления  
- **Документируйте** — «NSLayoutConstraint — центрирование аватарки с адаптивной шириной»

**Короткий девиз 2026**:
> NSLayoutConstraint — это **математическое правило** «этот край = тот край + 16».  
> В 2026 году **пиши через NSLayoutAnchor**, активируй массивом и используй **safeAreaLayoutGuide**.  
> Старый конструктор `NSLayoutConstraint(item:…)` — почти мёртв, оставь только для очень редких случаев.
