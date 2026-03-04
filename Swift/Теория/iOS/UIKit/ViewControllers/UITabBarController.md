#uitabbarcontroller #uitabbar #tab-bar #container-controller #navigation #uikit #ios #swift #tabbarappearance #accessibility #darkmode #dynamic-type

---
**(контроллер вкладок / таб-бар контроллер)**

**UITabBarController** — это **контейнерный контроллер** (container view controller) в [[UIKit]], который управляет **нижней панелью вкладок** ([[UITabBar]]) и набором дочерних контроллеров (`viewControllers`), каждый из которых соответствует одной вкладке.

Это **стандартный** и **самый популярный** способ реализации **нижней навигации** в iOS-приложениях 2026 года (Instagram, ВКонтакте, YouTube, TikTok, Spotify, Telegram и т.д.).

### 1. Ключевые моменты и архитектура

- **Всегда видна** панель вкладок внизу экрана (кроме редких случаев скрытия).
- Каждая вкладка — это **[[UITabBarItem]]** + связанный **[[UIViewController]]** (чаще всего обёрнутый в [[UINavigationController]]).
- Поддерживает до **5 вкладок** без проблем; при >5 появляется вкладка "More" (автоматически).
- Управляет **переключением** между экранами, **состоянием** (selectedIndex), **внешним видом** ([[UITabBarAppearance]]).

**Типичная структура (рекомендуемая в 2026):**

```text
UITabBarController (root)
├── UINavigationController (вкладка 1: "Главная")
│   └── HomeViewController
├── UINavigationController (вкладка 2: "Поиск")
│   └── SearchViewController
├── UINavigationController (вкладка 3: "Создать")
│   └── CreateViewController (часто модальный или без nav bar)
├── UINavigationController (вкладка 4: "Уведомления")
│   └── NotificationsViewController
└── UINavigationController (вкладка 5: "Профиль")
    └── ProfileViewController
```

### 2. Самый популярный и рекомендуемый паттерн 2026 года  
(UITabBarController + [[UINavigationController]] в каждой вкладке + [[UITabBarAppearance]] + [[Coordinator]])

```swift
import UIKit

class AppCoordinator {
    let window: UIWindow
    
    init(window: UIWindow) {
        self.window = window
    }
    
    func start() {
        let tabBarController = UITabBarController()
        
        // 1. Настраиваем глобальный вид таб-бара
        setupTabBarAppearance()
        
        // 2. Создаём вкладки
        tabBarController.viewControllers = [
            createNavController(for: HomeViewController(), title: "Главная", image: "house"),
            createNavController(for: SearchViewController(), title: "Поиск", image: "magnifyingglass"),
            createNavController(for: CreateViewController(), title: "Создать", image: "plus.circle"),
            createNavController(for: NotificationsViewController(), title: "Уведомления", image: "bell"),
            createNavController(for: ProfileViewController(), title: "Профиль", image: "person")
        ]
        
        // 3. Устанавливаем как root
        window.rootViewController = tabBarController
        window.makeKeyAndVisible()
    }
    
    private func createNavController(for rootVC: UIViewController, title: String, image: String) -> UIViewController {
        let nav = UINavigationController(rootViewController: rootVC)
        rootVC.navigationItem.title = title  // или large title
        nav.tabBarItem = UITabBarItem(title: title,
                                      image: UIImage(systemName: image),
                                      tag: 0)
        return nav
    }
    
    private func setupTabBarAppearance() {
        let appearance = UITabBarAppearance()
        appearance.configureWithOpaqueBackground()
        appearance.backgroundColor = .systemBackground
        
        // Цвет иконок и текста
        let normalAttributes: [NSAttributedString.Key: Any] = [
            .foregroundColor: UIColor.systemGray,
            .font: UIFont.systemFont(ofSize: 12, weight: .medium)
        ]
        
        let selectedAttributes: [NSAttributedString.Key: Any] = [
            .foregroundColor: UIColor.systemBlue,
            .font: UIFont.systemFont(ofSize: 12, weight: .semibold)
        ]
        
        appearance.stackedLayoutAppearance.normal.iconColor = .systemGray
        appearance.stackedLayoutAppearance.normal.titleTextAttributes = normalAttributes
        appearance.stackedLayoutAppearance.selected.iconColor = .systemBlue
        appearance.stackedLayoutAppearance.selected.titleTextAttributes = selectedAttributes
        
        // Для iOS 15+ — scroll edge
        if #available(iOS 15.0, *) {
            UITabBar.appearance().scrollEdgeAppearance = appearance
        }
        
        UITabBar.appearance().standardAppearance = appearance
    }
}
```

### 3. Лучшие практики UITabBarController в 2026 году

- **Всегда оборачивайте** каждый экран в `UINavigationController` — это стандарт 2026 (push внутри вкладки)
- **UITabBarAppearance** — глобально настраивайте через `UITabBar.appearance()` в `didFinishLaunchingWithOptions` или `scene(_:willConnectTo:)`
- **selectedIndex** / `selectedViewController` — используйте для программного переключения вкладок
- **tabBar.isHidden = true** — только если нужно временно скрыть (редко, лучше использовать custom tab bar)
- **Coordinator** — для сложной навигации и создания контроллеров (рекомендуется)
- **Доступность** — задавайте `accessibilityLabel` и `accessibilityHint` для `UITabBarItem`
- **Dynamic Type** — используйте `adjustsFontForContentSizeCategory = true` для текста в контроллерах
- **Тёмная тема** — системные цвета + `UITabBarAppearance` автоматически адаптируется
- **Документируйте** — пишите комментарий:

```swift
/// Настройка глобального вида таб-бара (цвета, шрифты, фон)
private func setupTabBarAppearance() {
    let appearance = UITabBarAppearance()
    // ...
}
```

**Короткий итог 2026**:
> **UITabBarController** — **контейнер** для **нижней навигации** с постоянной панелью вкладок.  
> В 2026 году:  
> - ключевые свойства — `viewControllers`, `selectedIndex`, `tabBarItem`  
> - самый популярный паттерн — UITabBarController + UINavigationController в каждой вкладке + UITabBarAppearance  
> - идеален для приложений с 3–5 независимыми разделами (Instagram, Spotify, Telegram)  
> - в SwiftUI — заменяется на `TabView`  
> - это **стандарт де-факто** для нижней навигации в UIKit-приложениях  
