**viewDidLayoutSubviews()** — это один из ключевых методов **жизненного цикла** [[UIViewController]] и [[UIView]] в [[UIKit]].

Он вызывается **каждый раз**, когда **подвиды (subviews)** контроллера или вью **были размещены** (layout) системой Auto Layout или вручную.

### Когда именно вызывается viewDidLayoutSubviews

| Событие / Действие                                                                       | viewDidLayoutSubviews вызывается?       | Примечание / важные детали                          |
| ---------------------------------------------------------------------------------------- | --------------------------------------- | --------------------------------------------------- |
| Первый показ экрана (после [[viewDidAppear]])                                            | **Да**                                  | После первой компоновки subviews                    |
| Изменение размеров экрана (поворот, split view, multitasking)                            | **Да**                                  | После вызова `viewWillTransition(to:with:)`         |
| Изменение размеров контроллера (например, при появлении/скрытии navigation bar, tab bar) | **Да**                                  | Часто при анимациях                                 |
| Вызов `setNeedsLayout()` / `layoutIfNeeded()`                                            | **Да** (если layout произошёл)          | После принудительной компоновки                     |
| Добавление / удаление subviews динамически                                               | **Да** (если вызван `layoutSubviews()`) | После `addSubview` / `removeFromSuperview` + layout |
| Изменение constraints (например, при анимации высоты)                                    | **Да**                                  | После обновления Auto Layout                        |
| Возврат с другого экрана (pop / dismiss)                                                 | **Да** (если размеры изменились)        | Не всегда, зависит от изменений layout              |

### Порядок вызовов (самая частая последовательность)

```text
1. viewDidLoad()
2. viewWillAppear(_:)
3. viewDidAppear(_:)
4. viewWillLayoutSubviews()   ← перед компоновкой
5. viewDidLayoutSubviews()    ← здесь ты обычно
6. (при изменении размеров) viewWillTransition(to:with:)
7. viewWillLayoutSubviews()
8. viewDidLayoutSubviews()
```

### Что обычно делают в viewDidLayoutSubviews в 2026 году

| Действие                                         | Почему именно здесь (а не в [[viewDidLoad]] / [[viewDidAppear]])            | Пример кода (современный стиль)                        |
| ------------------------------------------------ | --------------------------------------------------------------------------- | ------------------------------------------------------ |
| **Пересчёт размеров / позиций** subviews вручную | [[Auto Layout]] уже отработал → можно безопасно читать [[frame]]/[[bounds]] | `updateCustomLayout()`                                 |
| **Создание градиентов / масок / shadow paths**   | Нужно знать актуальные размеры после layout                                 | `layer.shadowPath = UIBezierPath(rect: bounds).cgPath` |
| **Настройка corner radius / masksToBounds**      | Зависит от финального размера                                               | `button.layer.cornerRadius = button.bounds.height / 2` |
| **Обновление scroll insets / content size**      | После изменения размеров navigation bar / safe area                         | `scrollView.contentInset = UIEdgeInsets(...)`          |
| **Пересчёт коллекции / таблицы** (reloadData)    | Если высота ячеек зависит от ширины экрана                                  | `collectionView.reloadData()`                          |
| **Добавление / обновление subviews динамически** | После первой компоновки или изменения размеров                              | `addGradientLayerIfNeeded()`                           |
| **Логирование / аналитика layout**               | Точное время, когда layout завершён                                         | `Analytics.trackLayoutComplete("Profile")`             |

### Самый современный паттерн 2026 ([[@MainActor]] + [[async]])

```swift
@MainActor
class ProfileViewController: UIViewController {
    
    private let profileImageView = UIImageView()
    private let nameLabel = UILabel()
    private lazy var gradientLayer = CAGradientLayer()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupSubviews()
    }
    
    private func setupSubviews() {
        profileImageView.translatesAutoresizingMaskIntoConstraints = false
        nameLabel.translatesAutoresizingMaskIntoConstraints = false
        
        view.addSubview(profileImageView)
        view.addSubview(nameLabel)
        
        // Auto Layout constraints
        NSLayoutConstraint.activate([
            profileImageView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            profileImageView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 40),
            profileImageView.widthAnchor.constraint(equalToConstant: 120),
            profileImageView.heightAnchor.constraint(equalToConstant: 120),
            
            nameLabel.topAnchor.constraint(equalTo: profileImageView.bottomAnchor, constant: 16),
            nameLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor)
        ])
        
        profileImageView.layer.cornerRadius = 60
        profileImageView.clipsToBounds = true
    }
    
    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
        
        // Здесь frame/bounds уже актуальны
        updateGradient()
        updateShadowPath()
    }
    
    private func updateGradient() {
        gradientLayer.frame = profileImageView.bounds
        gradientLayer.colors = [UIColor.systemBlue.cgColor, UIColor.systemPurple.cgColor]
        gradientLayer.startPoint = CGPoint(x: 0, y: 0)
        gradientLayer.endPoint = CGPoint(x: 1, y: 1)
        
        if gradientLayer.superlayer == nil {
            profileImageView.layer.insertSublayer(gradientLayer, at: 0)
        }
    }
    
    private func updateShadowPath() {
        profileImageView.layer.shadowPath = UIBezierPath(roundedRect: profileImageView.bounds, cornerRadius: 60).cgPath
        profileImageView.layer.shadowOpacity = 0.3
        profileImageView.layer.shadowRadius = 8
        profileImageView.layer.shadowOffset = CGSize(width: 0, height: 4)
    }
}
```

### Лучшие практики viewDidLayoutSubviews в Swift 2026

- **Всегда вызывай super.viewDidLayoutSubviews()** — в начале метода  
- **Не делай тяжёлую работу** — максимум 1–2 мс (иначе дропы кадров при анимациях/скролле)  
- **Не вызывай [[layoutIfNeeded]]() / setNeedsLayout()** внутри viewDidLayoutSubviews — это может привести к бесконечному циклу  
- **@MainActor** — весь контроллер или метод — на главном акторе  
- **Проверяй bounds.isEmpty** — иногда вызывается до первой компоновки  
- **Swift 6 strict concurrency** — все UI-операции — в [[@MainActor]]  
- **Документируйте** — пиши комментарий «viewDidLayoutSubviews — обновление градиентов и shadow path после layout»  

**Короткий девиз 2026**:
> «viewDidLayoutSubviews — это когда система уже **закончила размещать subviews** и ты можешь безопасно читать/использовать их актуальные frame/bounds.  
> Делай здесь всё, что зависит от **реальных размеров** после Auto Layout: градиенты, shadow path, corner radius, scroll insets.  
> Не делай ничего тяжёлого и не вызывай layout повторно — это место только для финальной настройки.»
