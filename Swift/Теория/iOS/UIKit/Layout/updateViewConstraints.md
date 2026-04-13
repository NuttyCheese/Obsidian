#updateviewconstraints #uikit #autolayout #constraints #view-lifecycle #ios #swift #dynamic-layout #traitcollection #size-classes #adaptive-ui

---
**(метод обновления ограничений представления)**

**updateViewConstraints()** — это один из ключевых методов жизненного цикла [[UIViewController]] и [[UIView]], который вызывается системой **перед тем, как [[Auto Layout]] начнёт раскладывать субвью** (layout pass).

Он даёт вам возможность **динамически изменять или добавлять constraints** в зависимости от текущих условий:

- изменения **size class** (iPhone → iPad, portrait → landscape)
- смены **темы** (light/dark)
- изменения **Dynamic Type** (размер текста)
- поворота экрана
- появления/исчезновения клавиатуры
- изменения размеров родительского контейнера

### Когда и почему вызывается updateViewConstraints()

Порядок вызовов (примерный жизненный цикл):

1. [[traitCollectionDidChange]]`(_:)` — если изменились черты (size class, тема, Dynamic Type)
2. [[viewWillTransition]]`(to:with:)` — при повороте экрана
3. `updateViewConstraints()` — **система вызывает** перед layout
4. [[viewWillLayoutSubviews]]`()`
5. [[viewDidLayoutSubviews]]`()`

**Важно:**  
- Метод может вызываться **много раз** за жизнь контроллера  
- Вы **обязаны** вызывать `super.updateViewConstraints()`  
- Не делайте здесь тяжёлых вычислений — это часть layout-пасс

### Самый популярный и рекомендуемый паттерн 2026 года

```swift
class AdaptiveProfileViewController: UIViewController {
    
    private let stackView = UIStackView()
    private let avatarImageView = UIImageView()
    private let nameLabel = UILabel()
    private let bioTextView = UITextView()
    
    // Храним constraints, которые будем менять
    private var stackAxisConstraint: NSLayoutConstraint!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        setupUI()
        updateConstraintsForCurrentTraits()
    }
    
    // Самое важное место — динамическое изменение constraints
    override func updateViewConstraints() {
        super.updateViewConstraints()
        
        // Удаляем старые constraints, если нужно
        if stackAxisConstraint != nil {
            stackAxisConstraint.isActive = false
        }
        
        // Адаптируем ось стека под size class
        if traitCollection.horizontalSizeClass == .regular {
            // iPad / landscape → горизонтальный стек
            stackView.axis = .horizontal
            stackAxisConstraint = stackView.heightAnchor.constraint(equalToConstant: 120)
        } else {
            // iPhone → вертикальный стек
            stackView.axis = .vertical
            stackAxisConstraint = stackView.widthAnchor.constraint(equalToConstant: 300)
        }
        
        stackAxisConstraint.isActive = true
        
        // Дополнительно: адаптация под Dynamic Type
        nameLabel.font = UIFont.preferredFont(forTextStyle: .title1)
        bioTextView.font = UIFont.preferredFont(forTextStyle: .body)
    }
    
    private func setupUI() {
        view.backgroundColor = .systemBackground
        
        stackView.axis = .vertical
        stackView.spacing = 16
        stackView.alignment = .center
        stackView.distribution = .fill
        stackView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(stackView)
        
        avatarImageView.contentMode = .scaleAspectFill
        avatarImageView.layer.cornerRadius = 50
        avatarImageView.clipsToBounds = true
        avatarImageView.backgroundColor = .systemGray5
        
        nameLabel.textAlignment = .center
        bioTextView.textAlignment = .center
        bioTextView.isEditable = false
        
        stackView.addArrangedSubview(avatarImageView)
        stackView.addArrangedSubview(nameLabel)
        stackView.addArrangedSubview(bioTextView)
        
        NSLayoutConstraint.activate([
            stackView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            stackView.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            avatarImageView.widthAnchor.constraint(equalToConstant: 100),
            avatarImageView.heightAnchor.constraint(equalToConstant: 100)
        ])
    }
    
    // Реакция на смену черт (тема, size class, Dynamic Type)
    override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
        super.traitCollectionDidChange(previousTraitCollection)
        
        // Вызываем updateViewConstraints вручную — это безопасно и рекомендуется
        setNeedsUpdateConstraints()
    }
}
```

### Почему именно updateViewConstraints(), а не viewDidLayoutSubviews()?

| Метод                          | Когда вызывается                     | Что можно менять безопасно                        | Рекомендация 2026              |
| ------------------------------ | ------------------------------------ | ------------------------------------------------- | ------------------------------ |
| `updateViewConstraints()`      | Перед layout pass (система вызывает) | Добавление/удаление/изменение **[[constraint]]s** | **Да** — основное место        |
| `viewWillLayoutSubviews()`     | Перед layout (вызывается много раз)  | Редко — только подготовка                         | Почти никогда                  |
| `viewDidLayoutSubviews()`      | После layout                         | Изменение [[frame]] вручную (редко)               | Только если нет constraints    |
| `traitCollectionDidChange(_:)` | При смене черт                       | Вызов `setNeedsUpdateConstraints()`               | Вызывать updateViewConstraints |

**Правило 2026 года:**  
Всё, что связано с **динамическими constraints** (size class, тема, Dynamic Type) — пиши в `updateViewConstraints()` и вызывай `setNeedsUpdateConstraints()` при необходимости.

### Лучшие практики updateViewConstraints() в 2026 году

- **Всегда вызывайте** `super.updateViewConstraints()` — иначе сломаете системные constraints
- **Используйте** `setNeedsUpdateConstraints()` вместо `setNeedsLayout()` — это правильно для constraints
- **Храните** изменяемые constraints в свойствах — чтобы их можно было деактивировать
- **Для Dynamic Type** — обновляйте шрифты через `preferredFont(forTextStyle:)` + `adjustsFontForContentSizeCategory = true`
- **Для тёмной темы** — используйте системные цвета или проверяйте `traitCollection.userInterfaceStyle`
- **Для SwiftUI** — аналогов нет ([[SwiftUI]] делает это автоматически) — `updateViewConstraints` нужен только в UIKit
- **Документируйте** — пишите комментарий:

```swift
override func updateViewConstraints() {
    super.updateViewConstraints()
    
    // Адаптируем layout под текущие size class и тему
    updateDynamicConstraints()
}
```

**Короткий итог 2026**:
> **updateViewConstraints()** — метод, который вызывается **перед раскладкой** и позволяет динамически менять **constraints** в зависимости от size class, темы, Dynamic Type.  
> Junior: "Место, где меняем положение элементов под разные экраны".  
> Middle: вызывается системой перед layout → пишем здесь адаптивные constraints.  
> Senior: всегда `super`, `setNeedsUpdateConstraints()` при смене traitCollection, избегайте тяжёлых вычислений, комбинируйте с `traitCollectionDidChange`.  
