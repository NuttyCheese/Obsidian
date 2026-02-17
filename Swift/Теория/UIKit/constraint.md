#uikit #Swift #ios 
**Constraint** (ограничение) — это основная единица **[[Auto Layout]]** в [[UIKit]]. Оно описывает **математическое отношение** между двумя элементами интерфейса (или между элементом и его контейнером/супервиу), которое система должна соблюдать при расчёте layout.

Каждое ограничение решает уравнение вида:

**item1.attribute1 = multiplier × item2.attribute2 + constant**

Примеры отношений:
- ведущий край кнопки = ведущий край родителя + 16
- ширина изображения = высота изображения × 1.5
- центр метки = центр родителя

### Основные способы создания Constraint в 2026 году

| Способ                                      | Уровень современности | Когда использовать в 2026             | Пример кода (коротко)                                                                                      |             |          |
| ------------------------------------------- | --------------------- | ------------------------------------- | ---------------------------------------------------------------------------------------------------------- | ----------- | -------- |
| **[[NSLayoutAnchor]]** (рекомендуемый)      | ★★★★★                 | Всегда, если пишешь UIKit вручную     | `view.topAnchor.constraint(equalTo: superview.topAnchor, constant: 20).isActive = true`                    |             |          |
| **[[NSLayoutConstraint]].activate([...])**  | ★★★★☆                 | Когда много констрейнтов сразу        | `NSLayoutConstraint.activate([v1.leadingAnchor.constraint(equalTo: v2.trailingAnchor, constant: 8), ...])` |             |          |
| **[[SnapKit]]**                             | ★★★★☆                 | Если хочешь читаемый и лаконичный код | `view.snp.makeConstraints { $0.top.equalToSuperview().offset(20) }`                                        |             |          |
| **[[SwiftUI]] .frame / .offset / .padding** | ★★★★★ (в SwiftUI)     | Если проект на SwiftUI                | `Text("Hello").frame(width: 100, height: 50)`                                                              |             |          |
| **VFL (Visual Format Language)**            | ★☆☆☆☆                 | Только legacy / очень редкие случаи   | `NSLayoutConstraint.constraints(withVisualFormat: "H:                                                      | -16-[v]-16- | ", ...)` |

### Самый популярный и рекомендуемый паттерн 2026 года (NSLayoutAnchor + activate)

```swift
class ProfileHeaderView: UIView {
    
    private let avatarImageView = UIImageView()
    private let nameLabel = UILabel()
    private let bioLabel = UILabel()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupSubviews()
        setupConstraints()
    }
    
    private func setupSubviews() {
        [avatarImageView, nameLabel, bioLabel].forEach {
            $0.translatesAutoresizingMaskIntoConstraints = false
            addSubview($0)
        }
        
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
            avatarImageView.heightAnchor.constraint(equalTo: avatarImageView.widthAnchor), // квадрат
            
            // Имя — справа от аватарки
            nameLabel.leadingAnchor.constraint(equalTo: avatarImageView.trailingAnchor, constant: 16),
            nameLabel.topAnchor.constraint(equalTo: avatarImageView.topAnchor),
            nameLabel.trailingAnchor.constraint(equalTo: trailingAnchor, constant: -16),
            
            // Био — под именем
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

### Лучшие практики Constraint в Swift 2026

- **translatesAutoresizingMaskIntoConstraints = false** — **обязательно** для всех programmatic вью  
- **NSLayoutConstraint.activate([...])** — активируй сразу массивом (одна операция, быстрее)  
- **safeAreaLayoutGuide / readableContentGuide** — вместо прямой привязки к superview  
- **priority** — используй .required (1000), .high (750), .low (250) для гибкости  
- **identifier** — задавай для всех важных констрейнтов (видно в отладчике)

```swift
let constraint = view.widthAnchor.constraint(equalToConstant: 200)
constraint.identifier = "profile-avatar-width"
constraint.priority = .defaultHigh
constraint.isActive = true
```

- **constant** — используй для отступов (положительные = внутрь, отрицательные = наружу)  
- **multiplier** — для пропорций (например, width = height × 1.618 — золотое сечение)  
- **@MainActor** — все операции с constraints — на главном акторе  
- **Swift 6 strict concurrency** — NSLayoutConstraint полностью [[Sendable]]-safe  
- **Документируйте** — пиши комментарий «NSLayoutConstraint — адаптивный layout с safe area»

**Короткий девиз 2026**:
> Constraint — это математическое правило «этот край = тот край + 16» или «ширина = высота × 1.5».  
> В 2026 году **единственный современный** способ — **NSLayoutAnchor + activate**.  
> SnapKit — для читаемости, SwiftUI — для новых проектов, VFL — уже почти мёртв.
