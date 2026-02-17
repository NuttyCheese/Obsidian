**viewDidLoad()** — это **самый первый** и **самый важный** метод жизненного цикла **[[UIViewController]]** в [[UIKit]], который вызывается **один раз** — сразу после того, как представление контроллера загружено в память (loadViewIfNeeded() → loadView() завершены).

### Когда именно вызывается viewDidLoad

| Событие / Действие                            | viewDidLoad вызывается?              | Сколько раз за жизнь контроллера   | Примечание / важные детали                         |
| --------------------------------------------- | ------------------------------------ | ---------------------------------- | -------------------------------------------------- |
| Первый показ экрана (push / present)          | **Да**                               | **Один раз**                       | После создания контроллера                         |
| Возврат с предыдущего экрана (pop / dismiss)  | **Нет**                              | —                                  | Контроллер уже существует                          |
| Переключение вкладки в [[UITabBarController]] | **Нет** (если контроллер уже создан) | Один раз при первом выборе вкладки | Только при первом создании                         |
| Поворот экрана / изменение размеров           | **Нет**                              | —                                  | [[viewDidLayoutSubviews]] / [[viewWillTransition]] |
| Приложение вернулось из фона                  | **Нет**                              | —                                  | [[viewWillAppear]] / [[viewDidAppear]]             |
| Контроллер добавлен как child                 | **Да** (при первом создании)         | Один раз                           | После addChild                                     |

### Порядок вызовов (самая частая последовательность)

```text
1. init / init(coder:) / init(nibName:bundle:)
2. loadView()           ← здесь создаётся self.view (если не переопределён)
3. viewDidLoad()        ← здесь ты обычно настраиваешь всё
4. viewWillAppear(_:)
5. viewDidAppear(_:)
```

### Что обычно делают в viewDidLoad в 2026 году

| Действие                                      | Почему именно здесь (а не в init или viewDidAppear) | Пример кода (современный стиль) |
|-----------------------------------------------|-----------------------------------------------------|---------------------------------|
| **Добавление subviews** (programmatic UI)     | view уже существует → можно добавлять subviews      | `setupSubviews()` |
| **Настройка Auto Layout constraints**         | view уже в иерархии → можно создавать constraints   | `NSLayoutConstraint.activate([...])` |
| **Создание и настройка UI-элементов**         | Один раз → не нужно повторять при каждом появлении  | `titleLabel = UILabel()` |
| **Подписка на уведомления / KVO** (долгосрочные) | Достаточно один раз (отписка в deinit / viewDidDisappear) | `NotificationCenter.default.addObserver(...)` |
| **Инициализация ViewModel / сервисов**        | Один раз при создании контроллера                   | `viewModel = ProfileViewModel()` |
| **Загрузка начальных данных** (если не асинхронно) | Экран ещё не виден → можно делать тяжёлую работу    | `loadInitialData()` (но лучше в viewDidAppear) |
| **Регистрация на делегаты / data sources**    | Один раз (UITableView, UICollectionView и т.д.)     | `tableView.dataSource = self` |
| **Настройка внешнего вида** (цвета, шрифты, стили) | Один раз → не нужно повторять                      | `view.backgroundColor = .systemBackground` |

### Самый современный паттерн 2026 ([[@MainActor]] + [[async]]/[[await]] + programmatic UI)

```swift
@MainActor
class ProfileViewController: UIViewController {
    
    private let viewModel: ProfileViewModel
    
    // UI-элементы
    private let profileImageView = UIImageView()
    private let nameLabel = UILabel()
    private let bioTextView = UITextView()
    
    init(viewModel: ProfileViewModel) {
        self.viewModel = viewModel
        super.init(nibName: nil, bundle: nil)
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        setupUI()
        setupBindings()
        
        // Можно начать лёгкую инициализацию
        // Но тяжёлые сетевые запросы лучше в viewDidAppear
    }
    
    private func setupUI() {
        view.backgroundColor = .systemBackground
        
        [profileImageView, nameLabel, bioTextView].forEach {
            $0.translatesAutoresizingMaskIntoConstraints = false
            view.addSubview($0)
        }
        
        profileImageView.contentMode = .scaleAspectFill
        profileImageView.layer.cornerRadius = 60
        profileImageView.clipsToBounds = true
        
        nameLabel.font = .preferredFont(forTextStyle: .title1)
        nameLabel.textAlignment = .center
        
        bioTextView.isEditable = false
        bioTextView.font = .preferredFont(forTextStyle: .body)
        
        NSLayoutConstraint.activate([
            profileImageView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 40),
            profileImageView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            profileImageView.widthAnchor.constraint(equalToConstant: 120),
            profileImageView.heightAnchor.constraint(equalToConstant: 120),
            
            nameLabel.topAnchor.constraint(equalTo: profileImageView.bottomAnchor, constant: 16),
            nameLabel.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            nameLabel.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20),
            
            bioTextView.topAnchor.constraint(equalTo: nameLabel.bottomAnchor, constant: 16),
            bioTextView.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            bioTextView.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20),
            bioTextView.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor, constant: -20)
        ])
    }
    
    private func setupBindings() {
        // Пример с Combine / @Published
        viewModel.$user
            .receive(on: DispatchQueue.main)
            .sink { [weak self] user in
                self?.nameLabel.text = user?.name
                self?.bioTextView.text = user?.bio
            }
            .store(in: &cancellables)
    }
    
    // Тяжёлые операции — лучше в viewDidAppear
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        
        Task {
            await viewModel.loadUser()
        }
    }
}
```

### Лучшие практики viewDidLoad в Swift 2026

- **Всегда вызывай super.viewDidLoad()** — в начале метода  
- **Создавай и настраивай UI здесь** — programmatic UI, constraints, стили  
- **Не делай тяжёлую работу** — сетевые запросы, сложные вычисления → переноси в [[viewDidAppear]] / [[Task]]  
- **@MainActor** — весь контроллер или метод — на главном акторе  
- **Отписывайся** в [[deinit]] / [[viewDidDisappear]] ([[KVO]], [[NotificationCenter]], [[Combine]])  
- **Swift 6 strict concurrency** — все UI-операции — в `@MainActor`  
- **Документируйте** — пиши комментарий «viewDidLoad — настройка UI и bindings один раз при создании контроллера»

**Короткий девиз 2026**:
> «viewDidLoad — это когда контроллер только что родился и пора **один раз** настроить весь UI: добавить subviews, constraints, bindings, стили.  
> Делай здесь всё, что нужно сделать **только один раз** за жизнь контроллера.  
> Тяжёлую работу, загрузку данных, аналитику — переноси в viewDidAppear.  
> Всегда вызывай super в начале.»
