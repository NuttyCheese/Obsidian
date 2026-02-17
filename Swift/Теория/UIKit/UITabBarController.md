**`UITabBarController`** — это **контейнерный контроллер** (container view controller), который управляет панелью вкладок (**[[UITabBar]]**) и связанными с ней [[UIViewController]].

🔑 Основные моменты:

- На экране всегда видна **панель вкладок** (`UITabBar`) внизу.
    
- Каждая вкладка ([[UITabBarItem]]) соответствует одному контроллеру (`UIViewController`).
    
- Пользователь может быстро переключаться между разделами приложения.
    

📱 Примеры из реальных приложений:

- Instagram (Лента, Поиск, Рилсы, Магазин, Профиль)
    
- ВКонтакте (Новости, Сервисы, Сообщения, Уведомления, Профиль)
    

---

## 2. Архитектура

`UITabBarController` → содержит массив `viewControllers`:

- `tabBarController.viewControllers = [vc1, vc2, vc3]`
    
- Каждый `vc` = экран приложения (например, через [[UINavigationController]]).
    

---

## 3. Пример использования

### Самый простой вариант:

```swift
let homeVC = HomeViewController()
homeVC.tabBarItem = UITabBarItem(title: "Главная", image: UIImage(systemName: "house"), tag: 0)

let profileVC = ProfileViewController()
profileVC.tabBarItem = UITabBarItem(title: "Профиль", image: UIImage(systemName: "person"), tag: 1)

let tabBarController = UITabBarController()
tabBarController.viewControllers = [homeVC, profileVC]

window?.rootViewController = tabBarController
window?.makeKeyAndVisible()
```

---

### Обычно комбинируют с `UINavigationController`

Чтобы в каждой вкладке была возможность **push-переходов**:

```swift
let homeVC = HomeViewController()
let homeNav = UINavigationController(rootViewController: homeVC)
homeNav.tabBarItem = UITabBarItem(title: "Главная", image: UIImage(systemName: "house"), tag: 0)

let profileVC = ProfileViewController()
let profileNav = UINavigationController(rootViewController: profileVC)
profileNav.tabBarItem = UITabBarItem(title: "Профиль", image: UIImage(systemName: "person"), tag: 1)

let tabBarController = UITabBarController()
tabBarController.viewControllers = [homeNav, profileNav]
```

👉 Теперь у каждой вкладки есть своя навигация.

---

## 4. Кастомизация

С помощью [[UITabBarAppearance]]:

```swift
let appearance = UITabBarAppearance()
appearance.configureWithOpaqueBackground()
appearance.backgroundColor = .systemGray6

// Настройка цвета текста и иконок
appearance.stackedLayoutAppearance.selected.iconColor = .systemBlue
appearance.stackedLayoutAppearance.selected.titleTextAttributes = [.foregroundColor: UIColor.systemBlue]

UITabBar.appearance().standardAppearance = appearance
if #available(iOS 15.0, *) {
    UITabBar.appearance().scrollEdgeAppearance = appearance
}
```

---

## 5. Работа с контроллером

Методы `UITabBarController`:

- `selectedIndex` → текущая активная вкладка
    
- `selectedViewController` → активный контроллер
    
- Делегат [[UITabBarControllerDelegate]] позволяет реагировать на выбор вкладки.
    

Пример:

```swift
tabBarController.selectedIndex = 1 // открыть вторую вкладку

if let currentVC = tabBarController.selectedViewController {
    print("Активный VC: \(currentVC)")
}
```

---

## 6. Когда использовать `UITabBarController`

✅ Подходит:

- Приложения с несколькими **независимыми разделами**.
    
- Приложения с "постоянной" нижней навигацией.
    

❌ Не подходит:

- Когда нужен **глубокий вложенный навигатор** (лучше `UINavigationController`).
    
- Когда количество разделов > 5 (начинается путаница, появляется "More").
    

---

## 7. Итог

- **`UITabBarController`** = главный контейнер для нижней навигации.
    
- Управляет `UITabBar` и массивом контроллеров.
    
- Часто комбинируется с `UINavigationController`.
    
- Кастомизируется через `UITabBarAppearance`.
    

---
