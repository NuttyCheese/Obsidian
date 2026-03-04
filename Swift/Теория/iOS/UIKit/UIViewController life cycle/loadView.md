**`loadView()`** — это один из ключевых методов жизненного цикла **[[UIViewController]]** в [[UIKit]].

Он отвечает за создание и установку свойства **`view`** контроллера.

### Когда и как вызывается loadView()

| Порядок в жизненном цикле контроллера        | Вызывается loadView()? | Что уже/ещё недоступно к этому моменту | Что обычно делают внутри |
| -------------------------------------------- | ---------------------- | -------------------------------------- | ------------------------ |
| [[init(nibName bundle)]] или [[init(coder)]] | Нет                    | view ещё [[nil]]                       | —                        |
| [[loadView]]()                               | **Да**                 | view ещё nil (но сейчас создаётся)     | Создание view вручную    |
| [[viewDidLoad]]()                            | Нет                    | view уже создан и загружен             | Настройка UI, outlets    |
| [[viewWillAppear]] / [[viewDidAppear]]       | Нет                    | view на экране                         | —                        |

**Важные факты 2026 года**:
- loadView() вызывается **лениво** (lazy) — только в момент первого обращения к `self.view` (если view ещё nil)
- Если вы **не переопределяете** loadView(), то [[UIKit]] автоматически:
  - Загружает view из Storyboard / nib (если контроллер создан через [[init(nibName bundle)]] или из [[Storyboard]])
  - Или создаёт пустой UIView (если контроллер создан чисто через init())
- Если вы **переопределяете** loadView() → **обязательно** установите `self.view = ...` вручную, иначе будет краш

### Самые популярные и рекомендуемые паттерны loadView() в 2026 году

#### 1. Самый частый — создание view программно (чистый код)

```swift
class ProfileViewController: UIViewController {
    
    override func loadView() {
        // Создаём корневой view вручную
        view = UIView()
        view.backgroundColor = .systemBackground
        
        // Можно сразу настроить базовые свойства
        view.isOpaque = true
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Теперь view уже существует → можно добавлять subviews, constraints
        setupUI()
    }
    
    private func setupUI() {
        // добавляем subviews, constraints, outlets и т.д.
    }
}
```

#### 2. Использование кастомного [[UIView]] как корневого (очень популярный паттерн)

```swift
class ProfileViewController: UIViewController {
    
    override func loadView() {
        // Вместо обычного UIView — наш кастомный
        view = ProfileRootView()
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Теперь view — это ProfileRootView
        if let rootView = view as? ProfileRootView {
            rootView.configure(with: viewModel)
        }
    }
}

// Сам кастомный корневой view
class ProfileRootView: UIView {
    // все subviews, constraints, стили здесь
    // можно сделать @IBOutlet или программно
}
```

#### 3. Редкий, но важный случай — отключение автоматической загрузки nib

```swift
class CustomViewController: UIViewController {
    
    override func loadView() {
        // Мы хотим полностью контролировать создание view
        // и НЕ загружать ничего из nib/Storyboard
        view = UIView(frame: UIScreen.main.bounds)
        view.backgroundColor = .systemGroupedBackground
    }
    
    // Важно: если контроллер создаётся из Storyboard,
    // то loadView() НЕ вызывается автоматически — нужно переопределить и вызвать вручную
}
```

### Чего **НИКОГДА** нельзя делать в loadView() в 2026 году

| Запрещённое действие                                          | Почему нельзя делать                                  | Куда перенести                           |
| ------------------------------------------------------------- | ----------------------------------------------------- | ---------------------------------------- |
| Обращение к [[@IBOutlet]] / IBAction                          | Они ещё не установлены (nib не загружен)              | `viewDidLoad()`                          |
| Добавление subviews и constraints                             | Лучше делать в `viewDidLoad()` или в кастомном UIView | `viewDidLoad()` или `awakeFromNib()`     |
| Доступ к `self.navigationController`, `self.tabBarController` | Они могут быть ещё nil                                | `viewWillAppear()` или `viewDidAppear()` |
| Сетевые запросы, тяжёлые вычисления                           | Задерживает появление контроллера                     | `viewDidLoad()` или позже                |
| Изменение navigationItem, tabBarItem                          | Лучше делать после того, как контроллер в иерархии    | `viewDidLoad()` или `viewWillAppear()`   |

### Лучшие практики loadView() в Swift 2026

- **Переопределяйте** loadView() **только** если:
  - хотите полностью программно создать view (без nib/Storyboard)
  - хотите использовать кастомный корневой UIView (ProfileRootView, RootScrollView и т.д.)
- **Никогда** не забывайте присвоить `self.view = ...` — иначе краш
- **Вызывайте** `super.loadView()` — если не переопределяете полностью (редко нужно)
- **Для [[SwiftUI]]** — `loadView()` **не существует** (UIViewRepresentable / UIHostingController используют другой жизненный цикл)
- **Для Storyboard** — loadView() **не вызывается** (UIKit сам загружает view из Storyboard)
- **Документируйте** — пишите комментарий «loadView — программное создание корневого view контроллера»

**Короткий итог 2026**:
> `loadView()` — это метод, в котором вы **создаёте и устанавливаете** свойство `view` контроллера.  
> В 2026 году:  
> - переопределяйте **только** для программного создания view или кастомного корневого UIView  
> - **обязательно** делайте `self.view = ...`  
> - **нельзя** обращаться к IBOutlet, делать layout, добавлять subviews  
> - если используете Storyboard — loadView() **не нужен**  
> Это **редко переопределяемый**, но **очень мощный** метод для полного контроля над созданием UI контроллера.
