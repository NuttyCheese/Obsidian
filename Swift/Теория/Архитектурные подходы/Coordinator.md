**Coordinator** — это один из самых популярных и полезных архитектурных паттернов в iOS-разработке 2025–2026 годов.

Он решает сразу несколько очень болезненных проблем классического [[MVC (Model-View-Controller) Architecture|MVC]] / [[MVVM (Model-View-ViewModel) Architecture|MVVM]]:

- **[[UIViewController]] знает слишком много** (навигация, создание других контроллеров, бизнес-логика)
- сильная **связанность** между экранами
- сложность тестирования навигации
- дублирование кода переходов
- трудно поддерживать глубокие навигационные стеки и модальные сценарии

Coordinator берёт на себя ответственность за **навигацию** и **координацию потоков экранов**, оставляя контроллерам только отображение и локальную логику.

### Основные идеи Coordinator-паттерна

1. **Координатор ≠ контроллер**  
   Это отдельный объект (обычно класс), который:
   - знает, какие экраны должны показываться в каком порядке
   - создаёт и конфигурирует ViewController'ы
   - управляет навигацией (push, present, pop, dismiss, set root и т.д.)
   - передаёт данные между экранами (через модель или closure)

2. **Иерархия координаторов**  
   Обычно строится дерево:
   - AppCoordinator (самый верхний — управляет окном / сценой)
   - TabBarCoordinator / AuthCoordinator / MainFlowCoordinator
   - вложенные: ProfileCoordinator, SettingsCoordinator, OnboardingCoordinator и т.д.

3. **Слабая связанность**  
   ViewController почти ничего не знает о других экранах и о том, как перейти дальше.

### Самые популярные варианты реализации в 2026 году

#### 1. Классический Coordinator с протоколом (самый распространённый)

```swift
protocol Coordinator: AnyObject {
    var childCoordinators: [Coordinator] { get set }
    var navigationController: UINavigationController { get set } // или другой контейнер
    
    func start()
}

class AppCoordinator: Coordinator {
    var childCoordinators: [Coordinator] = []
    var navigationController: UINavigationController
    
    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }
    
    func start() {
        let mainCoordinator = MainTabBarCoordinator(navigationController: navigationController)
        childCoordinators.append(mainCoordinator)
        mainCoordinator.start()
    }
}

class MainTabBarCoordinator: Coordinator {
    var childCoordinators: [Coordinator] = []
    var navigationController: UINavigationController
    
    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }
    
    func start() {
        let tabBarController = UITabBarController()
        
        let homeCoordinator = HomeCoordinator()
        homeCoordinator.start()
        let homeVC = homeCoordinator.rootViewController
        homeVC.tabBarItem = UITabBarItem(title: "Главная", image: UIImage(systemName: "house"), tag: 0)
        
        // ... другие вкладки
        
        tabBarController.viewControllers = [homeVC /*, ... */]
        navigationController.setViewControllers([tabBarController], animated: false)
    }
}
```

#### 2. Coordinator + Flow (очень популярный в 2025–2026)

```swift
protocol FlowCoordinator: Coordinator {
    var onFinish: (() -> Void)? { get set }
}

class AuthCoordinator: FlowCoordinator {
    var childCoordinators: [Coordinator] = []
    var navigationController: UINavigationController
    var onFinish: (() -> Void)?
    
    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }
    
    func start() {
        let loginVC = LoginViewController()
        loginVC.onLoginSuccess = { [weak self] user in
            self?.onFinish?()
        }
        navigationController.setViewControllers([loginVC], animated: false)
    }
}
```

#### 3. Router внутри Coordinator (очень чистый вариант)

```swift
protocol Router: AnyObject {
    func showLogin()
    func showRegistration()
    func showMainTabBar()
}

class AuthRouter: Router {
    weak var navigationController: UINavigationController?
    
    func showLogin() {
        let vc = LoginViewController()
        navigationController?.setViewControllers([vc], animated: true)
    }
    
    // ...
}
```

### Преимущества Coordinator в 2026 году

- **Навигация вынесена** из контроллеров → контроллеры становятся «глупыми»
- **Легко тестировать** навигационные потоки (можно мокать Router / Coordinator)
- **Поддержка сложных сценариев** — deep linking, восстановление стека, модальные потоки, tab bar + navigation
- **Чистый код** — нет цепочек `presentingViewController?.presentingViewController?.dismiss`
- **Легко добавлять/удалять экраны** — всё в одном месте

### Недостатки и компромиссы

- Дополнительный слой абстракции → больше кода на старте
- Нужно следить за **слабой связанностью** (weak ссылки, избегать retain cycle)
- Иногда кажется over-engineering для маленьких приложений

### Альтернативы Coordinator в 2026 году

| Подход                                                                                     | Когда лучше Coordinator                       | Плюсы                         | Минусы                                        |
| ------------------------------------------------------------------------------------------ | --------------------------------------------- | ----------------------------- | --------------------------------------------- |
| **SwiftUI + NavigationStack / NavigationSplitView**                                        | Новые проекты на [[SwiftUI]]                  | Декларативно, меньше кода     | Пока хуже поддержка сложных модальных потоков |
| **Router + Dependency Injection**                                                          | Хочется минимизировать количество классов     | Легче, чем полный Coordinator | Меньше изоляции навигации                     |
| **MVVM + [[Closure]] / Publisher**                                                         | Простые приложения, переходы не очень сложные | Минимум кода                  | Контроллеры начинают разрастаться             |
| **[[VIPER Architecture\|VIPER]] / [[Clean Swift (VIP) Architecture\|Clean Architecture]]** | Очень большие корпоративные проекты           | Максимальная изоляция         | Слишком много boilerplate                     |
| **Coordinator + SwiftUI Hosting**                                                          | Смешанный [[UIKit]] + SwiftUI проект          | Лучшее из двух миров          | Нужно следить за двумя мирами                 |

### Короткий итог 2026

> **Coordinator** — паттерн, который **выносит навигацию и создание экранов** из UIViewController'ов в отдельные объекты.  
> В 2026 году это:  
> - один из **самых популярных** архитектурных паттернов в UIKit  
> - фактически **стандарт де-факто** для приложений со сложной навигацией  
> - особенно полезен при: deep linking, модальных потоках, tab bar + stack, восстановлении состояния  
> - в чистом SwiftUI почти не нужен (NavigationStack делает многое сам)  
> - но в **UIKit** и **смешанных** проектах — остаётся одним из лучших решений  
