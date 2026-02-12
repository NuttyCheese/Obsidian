## 1. Что это такое

**`UITabBarControllerDelegate`** — это протокол, который позволяет **реагировать на действия пользователя** в [[UITabBarController]].

С его помощью можно:

- узнавать, когда пользователь выбрал вкладку,
    
- запрещать переход к определённой вкладке,
    
- отслеживать завершение кастомной анимации при смене контроллера.
    

---

## 2. Основные методы протокола

```swift
protocol UITabBarControllerDelegate : NSObjectProtocol {

    // 🔹 Вызывается перед тем, как будет выбран контроллер
    optional func tabBarController(
        _ tabBarController: UITabBarController,
        shouldSelect viewController: UIViewController
    ) -> Bool

    // 🔹 Вызывается после того, как контроллер был выбран
    optional func tabBarController(
        _ tabBarController: UITabBarController,
        didSelect viewController: UIViewController
    )

    // 🔹 Вызывается, когда начинается кастомный переход (если используется)
    optional func tabBarController(
        _ tabBarController: UITabBarController,
        animationControllerForTransitionFrom fromVC: UIViewController,
        to toVC: UIViewController
    ) -> UIViewControllerAnimatedTransitioning?

    // 🔹 Вызывается при завершении настройки `UITabBarController`
    optional func tabBarControllerSupportedInterfaceOrientations(
        _ tabBarController: UITabBarController
    ) -> UIInterfaceOrientationMask
}
```

---

## 3. Пример использования

### Назначаем делегата:

```swift
class MainTabBarController: UITabBarController, UITabBarControllerDelegate {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        self.delegate = self
    }
}
```

### Реагируем на выбор вкладки:

```swift
func tabBarController(
    _ tabBarController: UITabBarController,
    didSelect viewController: UIViewController
) {
    print("Выбрана вкладка: \(viewController.title ?? "Без названия")")
}
```

### Запрещаем открывать определённую вкладку:

```swift
func tabBarController(
    _ tabBarController: UITabBarController,
    shouldSelect viewController: UIViewController
) -> Bool {
    if viewController is RestrictedViewController {
        print("Эта вкладка недоступна 🚫")
        return false
    }
    return true
}
```

---

## 4. Где это применимо

- Если нужно **контролировать логику переключения вкладок** (например, запрещать неавторизованным пользователям переходить в «Профиль»).
    
- Если надо **логировать поведение пользователя** (какие вкладки чаще всего выбираются).
    
- Если используются **кастомные анимации** перехода между вкладками.
    

---

## 5. Итог

- `UITabBarControllerDelegate` = контроль поведения таб-бара.
    
- Ключевые методы:
    
    - `shouldSelect` → решает, можно ли перейти.
        
    - `didSelect` → сообщает, что переход выполнен.
        
    - `animationControllerForTransition…` → кастомные анимации.
        

---
