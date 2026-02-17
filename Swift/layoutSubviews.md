**`layoutSubviews()`** — это метод жизненного цикла [[UIView]], который система вызывает, когда нужно **пересчитать и разместить** все подпредставления (subviews) внутри текущего представления.

Это **единственное место**, где ты имеешь право и должен **вручную** управлять расположением и размерами своих subviews, если не используешь только [[Auto Layout]].

### Когда система вызывает `layoutSubviews()` (2026 актуально)

| Событие / Действие                                              | Вызывает `layoutSubviews()`? | Примечание / важные детали                         |
| --------------------------------------------------------------- | ---------------------------- | -------------------------------------------------- |
| Изменение [[bounds]] / [[frame]] текущей вью                    | **Да**                       | Самый частый триггер                               |
| `setNeedsLayout()` → следующий [[runloop]]                      | **Да**                       | Отложенный вызов                                   |
| `layoutIfNeeded()` вызван вручную                               | **Да** (немедленно)          | Принудительный пересчёт                            |
| Добавление / удаление subview                                   | **Да**                       | После `addSubview` / `removeFromSuperview`         |
| Изменение constraints (после `activate` или изменения constant) | **Да** (обычно отложено)     | Если был `setNeedsLayout()` или `layoutIfNeeded()` |
| Поворот устройства / изменение размеров экрана                  | **Да**                       | После `viewWillTransition`                         |
| Первый показ вью (после [[viewDidLoad]])                        | **Да**                       | Обычно один раз                                    |

### Самый важный принцип 2026 года

> **Никогда** не вызывай [[layoutIfNeeded]]() или `setNeedsLayout()` **внутри** `layoutSubviews()` на ту же вью — это **бесконечный цикл** и краш приложения.

Правильная последовательность:

```
Изменение условий (constraints, frame, добавление subview)
          ↓
setNeedsLayout()  ← если не срочно
          ↓
(следующий кадр)
          ↓
layoutSubviews()  ← здесь ты размещаешь subviews вручную
```

### Самый популярный и правильный паттерн 2026 года (программный layout)

```swift
final class ProfileHeaderView: UIView {
    
    private let avatarImageView = UIImageView()
    private let nameLabel = UILabel()
    private let bioLabel = UILabel()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupSubviews()
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
    
    override func layoutSubviews() {
        super.layoutSubviews()  // всегда первым!
        
        // 1. Размещаем аватарку по центру сверху
        let avatarSize: CGFloat = min(bounds.width * 0.25, 80)
        avatarImageView.frame = CGRect(
            x: (bounds.width - avatarSize) / 2,
            y: safeAreaInsets.top + 16,
            width: avatarSize,
            height: avatarSize
        )
        
        // 2. Имя под аватаркой
        let nameSize = nameLabel.sizeThatFits(CGSize(width: bounds.width - 32, height: .greatestFiniteMagnitude))
        nameLabel.frame = CGRect(
            x: 16,
            y: avatarImageView.frame.maxY + 12,
            width: bounds.width - 32,
            height: nameSize.height
        )
        
        // 3. Био под именем
        let bioSize = bioLabel.sizeThatFits(CGSize(width: bounds.width - 32, height: .greatestFiniteMagnitude))
        bioLabel.frame = CGRect(
            x: 16,
            y: nameLabel.frame.maxY + 8,
            width: bounds.width - 32,
            height: bioSize.height
        )
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

### Лучшие практики `layoutSubviews()` в Swift 2026

- **Всегда вызывай `super.layoutSubviews()` первым** — иначе ломается стандартный [[Auto Layout]]  
- **Не вызывай `layoutIfNeeded()` или `setNeedsLayout()` на себя** — бесконечный цикл  
- **Используй `bounds` / `safeAreaInsets`** — это актуальные размеры на момент вызова  
- **Избегай тяжёлых вычислений** — `layoutSubviews()` может вызываться очень часто (при скролле, повороте, анимации)  
- **Для чистого Auto Layout** — `layoutSubviews()` обычно не переопределяют (система сама всё сделает)  
- **Для гибридного подхода** — Auto Layout + ручное размещение нескольких элементов — переопределяют  
- **[[@MainActor]]** — весь кастомный UIView — на главном акторе  
- **[[Swift]] 6 strict concurrency** — `layoutSubviews()` вызывается на главном потоке → безопасно  
- **Документируйте** — пиши комментарий «layoutSubviews — ручное размещение аватарки и текстов под ней»

**Короткий девиз 2026**:
> `layoutSubviews()` — это **единственное место**, где ты имеешь право **вручную** расставлять frame своих subviews.  
> Вызывается системой после `setNeedsLayout` или изменения размеров.  
> Всегда вызывай `super` первым и **никогда** не вызывай `layoutIfNeeded()` внутри него.
