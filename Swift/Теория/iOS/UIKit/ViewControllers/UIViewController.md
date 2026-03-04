#uiviewcontroller #uikit #lifecycle #navigation #view-lifecycle #ios #swift #mvc #mvvm #container-controller #traitcollection #accessibility

---
**(контроллер представления / контроллер экрана)**

**UIViewController** — это **центральный класс** в [[UIKit]], который управляет **одним экраном** (или его значимой частью) в [[iOS]]-приложении.

Он отвечает за:

- создание и управление иерархией view (основной `view`)
- **жизненный цикл** экрана (загрузка, появление, исчезновение, поворот, изменение темы и т.д.)
- реакцию на события пользователя (нажатия, жесты, клавиатура)
- координацию **навигации** (push, present, pop, dismiss)
- адаптацию интерфейса под разные устройства и режимы (size class, тёмная/светлая тема, Dynamic Type, RTL)

Проще говоря:  
**UIViewController = логика экрана + управление его содержимым.**

Без него в UIKit невозможно построить полноценный экранный интерфейс.

### 1. Основные обязанности UIViewController (от junior к senior)

**Junior уровень — что делает контроллер на практике**

```swift
class HomeViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        
        let label = UILabel()
        label.text = "Добро пожаловать!"
        label.textAlignment = .center
        label.font = .systemFont(ofSize: 24, weight: .bold)
        label.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(label)
        
        NSLayoutConstraint.activate([
            label.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            label.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }
}
```

**Middle уровень — жизненный цикл и навигация**

Жизненный цикл (в порядке вызова):

1. [[init]]`(nibName:bundle:)` / `init(coder:)` — инициализация
2. [[loadView]]`()` — создание `view` (если не из [[XIB]]/[[Storyboard]])
3. [[viewDidLoad]]`()` — **один раз**, после загрузки view (самое частое место для setup)
4. [[viewWillAppear]]`(_:)` — перед появлением (может вызываться много раз)
5. [[viewDidAppear]]`(_:)` — после появления
6. [[viewWillDisappear]]`(_:)` — перед исчезновением
7. [[viewDidDisappear]]`(_:)` — после исчезновения
8. [[deinit]] — когда контроллер уничтожается (освобождение ресурсов)

**Навигация (самые частые способы):**

```swift
// Push в навигационный стек
navigationController?.pushViewController(detailVC, animated: true)

// Модальное представление
present(detailVC, animated: true)

// Pop назад
navigationController?.popViewController(animated: true)

// Dismiss модального
dismiss(animated: true)

// Замена root (редко)
navigationController?.setViewControllers([newRootVC], animated: true)
```

**Senior уровень — глубокая кастомизация и оптимизация**

1. **[[traitCollectionDidChange]](_:)** — адаптация под тёмную тему, size class, Dynamic Type

```swift
override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
    super.traitCollectionDidChange(previousTraitCollection)
    
    if traitCollection.userInterfaceStyle != previousTraitCollection?.userInterfaceStyle {
        updateColorsForCurrentStyle()
    }
    
    if traitCollection.preferredContentSizeCategory != previousTraitCollection?.preferredContentSizeCategory {
        updateFontsForDynamicType()
    }
}
```

2. **[[updateViewConstraints]]()** — кастомные constraints (вызывается перед layout)

```swift
override func updateViewConstraints() {
    super.updateViewConstraints()
    
    if traitCollection.horizontalSizeClass == .regular {
        // iPad — две колонки
        stackView.axis = .horizontal
    } else {
        stackView.axis = .vertical
    }
}
```

3. **Дочерние контроллеры** (container view controllers)

```swift
let childVC = ChildViewController()
addChild(childVC)
childVC.view.translatesAutoresizingMaskIntoConstraints = false
containerView.addSubview(childVC.view)
NSLayoutConstraint.activate([
    childVC.view.topAnchor.constraint(equalTo: containerView.topAnchor),
    childVC.view.bottomAnchor.constraint(equalTo: containerView.bottomAnchor),
    childVC.view.leadingAnchor.constraint(equalTo: containerView.leadingAnchor),
    childVC.view.trailingAnchor.constraint(equalTo: containerView.trailingAnchor)
])
childVC.didMove(toParent: self)
```

4. **Переопределение loadView()** (редко, но мощно)

```swift
override func loadView() {
    // Полностью кастомная view вместо UIView
    view = CustomBackgroundView()
}
```

5. **Оптимизация памяти** — [[didReceiveMemoryWarning]]`()`

```swift
override func didReceiveMemoryWarning() {
    super.didReceiveMemoryWarning()
    
    // Очистка кэша изображений, тяжёлых данных
    imageCache.removeAllObjects()
}
```

### 2. Самые частые паттерны 2026 года

1. **[[MVVM (Model-View-ViewModel) Architecture|MVVM]] + [[Combine]] + [[UIKit]]** (самый популярный сейчас)

```swift
class ProfileViewController: UIViewController {
    private let viewModel = ProfileViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var nameLabel: UILabel = { ... }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        viewModel.$userName
            .receive(on: DispatchQueue.main)
            .assign(to: \.text, on: nameLabel)
            .store(in: &cancellables)
    }
}
```

2. **[[Coordinator]] + UIKit** (для сложной навигации)

```swift
class ProfileCoordinator: Coordinator {
    func start() {
        let vc = ProfileViewController()
        vc.coordinator = self
        navigationController.pushViewController(vc, animated: true)
    }
    
    func showEditProfile() {
        let editVC = EditProfileViewController()
        navigationController.pushViewController(editVC, animated: true)
    }
}
```

3. **[[SwiftUI]] Hosting в UIKit**

```swift
let hostingVC = UIHostingController(rootView: ProfileSwiftUIView(viewModel: viewModel))
addChild(hostingVC)
hostingVC.view.frame = containerView.bounds
containerView.addSubview(hostingVC.view)
hostingVC.didMove(toParent: self)
```

### 3. Лучшие практики UIViewController в 2026 году

- **viewDidLoad()** — настройка UI, подписки, начальные данные  
- **viewWillAppear(_:)** — обновление данных, которые могут измениться при возврате  
- **traitCollectionDidChange(_:)** — адаптация под тему, size class, Dynamic Type  
- **deinit** — освобождение ресурсов (invalidate таймеры, remove observers)  
- **weak self** в замыканиях — избегать retain cycle  
- **[[Combine]] / [[async]]/[[await]]** — для реактивности и сетевых запросов  
- **Доступность** — `accessibilityLabel`, `accessibilityHint`, `adjustsFontForContentSizeCategory`  
- **Документируйте** — пишите комментарий:

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    // Настройка UI и подписок
    setupUI()
    bindViewModel()
}
```

**Короткий итог 2026**:
> **UIViewController** — **центральный объект** UIKit, управляющий одним экраном и его жизненным циклом.  
> В 2026 году:  
> - ключевые методы — `viewDidLoad`, `viewWill/DidAppear`, `traitCollectionDidChange`, `deinit`  
> - самый популярный паттерн — MVVM + Combine + Coordinator  
> - идеален для сложной навигации, адаптивного UI, контейнеров  
> - в SwiftUI — частично заменяется на `View`, но UIKit всё ещё доминирует в legacy и смешанных проектах  
> - это **основа** любого iOS-приложения на UIKit  
