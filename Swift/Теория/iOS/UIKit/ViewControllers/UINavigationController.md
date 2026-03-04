#uinavigationcontroller #uikit #navigation #stack #push-pop #navbar #ios #swift #container-controller #navigationbarappearance #swipe-back #large-title #ios-13

---
**(навигационный контроллер / контроллер навигации / нав-контроллер)**

**UINavigationController** — это **контейнерный контроллер** (container view controller) в [[UIKit]], который управляет **стеком экранов** ([[FILO|LIFO]] — Last In, First Out) и предоставляет пользователю **иерархическую навигацию** вперёд-назад.

Он автоматически:

- добавляет сверху **[[UINavigationBar]]** с заголовком, кнопкой «Назад» и возможностью кастомных кнопок
- поддерживает **swipe-to-go-back** (свайп от левого края экрана)
- управляет **переходами** (push / pop / popToRoot)
- позволяет кастомизировать внешний вид бара через **[[UINavigationBarAppearance]]**

Это **самый популярный и стандартный** способ организации навигации в [[iOS]]-приложениях 2026 года (почти все приложения с несколькими экранами используют его).

### 1. Основные принципы работы (от junior к senior)

**Junior уровень — как это выглядит на практике**

```swift
// Создаём главный навигатор с первым экраном
let homeVC = HomeViewController()
let navController = UINavigationController(rootViewController: homeVC)

// Устанавливаем как root окна
window?.rootViewController = navController
window?.makeKeyAndVisible()
```

**Переход вперёд (push)**

```swift
let detailVC = DetailViewController()
navigationController?.pushViewController(detailVC, animated: true)
```

**Возврат назад (pop)**

```swift
navigationController?.popViewController(animated: true)         // на один назад
navigationController?.popToRootViewController(animated: true)   // сразу на главный экран
```

**Senior уровень — глубокая кастомизация и тонкая настройка**

1. **UINavigationBarAppearance** — глобальная настройка (iOS 13+)

```swift
func setupNavigationBarAppearance() {
    let appearance = UINavigationBarAppearance()
    appearance.configureWithOpaqueBackground()
    appearance.backgroundColor = .systemBackground
    appearance.titleTextAttributes = [
        .font: UIFont.systemFont(ofSize: 17, weight: .semibold),
        .foregroundColor: UIColor.label
    ]
    
    // Большой заголовок (large title)
    appearance.largeTitleTextAttributes = [
        .font: UIFont.systemFont(ofSize: 34, weight: .bold),
        .foregroundColor: UIColor.label
    ]
    
    // Прозрачный бар при скролле (scroll edge)
    let scrollAppearance = UINavigationBarAppearance()
    scrollAppearance.configureWithTransparentBackground()
    scrollAppearance.backgroundEffect = UIBlurEffect(style: .systemMaterial)
    
    UINavigationBar.appearance().standardAppearance = appearance
    UINavigationBar.appearance().scrollEdgeAppearance = scrollAppearance
    UINavigationBar.appearance().compactAppearance = appearance
}
```

2. **Кастомная кнопка «Назад»**

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    // Скрыть стандартную кнопку "Назад" и поставить свою
    navigationItem.hidesBackButton = true
    
    let backButton = UIBarButtonItem(title: "Назад", style: .plain, target: self, action: #selector(customBack))
    navigationItem.leftBarButtonItem = backButton
}

@objc private func customBack() {
    navigationController?.popViewController(animated: true)
}
```

3. **Обработка swipe-to-go-back (interactive pop)**

```swift
// Отключить swipe-to-go-back (редко)
navigationController?.interactivePopGestureRecognizer?.isEnabled = false

// Или кастомизировать через UINavigationControllerDelegate
extension ViewController: UINavigationControllerDelegate {
    func navigationController(_ navigationController: UINavigationController,
                              animationControllerFor operation: UINavigationController.Operation,
                              from fromVC: UIViewController,
                              to toVC: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        // Кастомная анимация push/pop
        return CustomTransitionAnimator()
    }
}
```

4. **Large Title + Scroll Edge**

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    navigationItem.title = "Профиль"
    navigationItem.largeTitleDisplayMode = .always  // .automatic / .never / .inline
}
```

### 2. Лучшие практики UINavigationController в 2026 году

- **Всегда** оборачивайте первый экран в `UINavigationController` как root
- **Комбинируйте** с [[UITabBarController]] — каждая вкладка получает свой независимый стек
- **UINavigationBarAppearance** — глобально настраивайте в [[didFinishLaunchingWithOptions]] / `scene(_:willConnectTo:)`
- **largeTitleDisplayMode** — используйте `.always` для современных приложений
- **hidesBarsOnSwipe / hidesBarsWhenVerticallyCompact** — для полноэкранного контента
- **interactivePopGestureRecognizer** — оставляйте включённым (стандартный жест назад)
- **[[Coordinator]]** — для сложной навигации и создания контроллеров (рекомендуется)
- **Доступность** — `accessibilityLabel` для кнопок, `accessibilityHint` для навигации
- **Документируйте** — пишите комментарий:

```swift
/// Главный навигационный контроллер приложения с кастомным стилем
let navController = UINavigationController(rootViewController: HomeViewController())
```

**Короткий итог 2026**:
> **UINavigationController** — **контейнер** для **стека экранов** с автоматической навигационной панелью и поддержкой push/pop.  
> В 2026 году:  
> - ключевые методы — `pushViewController`, `popViewController`, `setViewControllers`  
> - самый популярный паттерн — UINavigationController + UITabBarController + UINavigationBarAppearance  
> - идеален для иерархической навигации (Главная → Категория → Детали → Корзина)  
> - в [[SwiftUI]] — заменяется на `NavigationStack`  
> - это **основа** большинства iOS-приложений на UIKit  
