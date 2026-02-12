## 1. Что такое `UITabBarItem`

**`UITabBarItem`** — это объект, который описывает **одну вкладку** в [[UITabBar]].  
Каждая вкладка в [[UITabBarController]] обязательно имеет свой `UITabBarItem`.

🔑 Вкладка состоит из:

- Заголовка (`title`)
    
- Иконки (`image`)
    
- Выбранной иконки (`selectedImage`)
    
- Дополнительно: бейдж (`badgeValue`)
    

---

## 2. Создание `UITabBarItem`

### Вариант 1 — через инициализатор

```swift
let item = UITabBarItem(
    title: "Главная",
    image: UIImage(systemName: "house"),
    selectedImage: UIImage(systemName: "house.fill")
)
```

### Вариант 2 — через `tag`

```swift
let item = UITabBarItem(title: "Профиль", image: UIImage(systemName: "person"), tag: 1)
```

### Вариант 3 — прямо у контроллера

```swift
let homeVC = HomeViewController()
homeVC.tabBarItem = UITabBarItem(title: "Главная", image: UIImage(systemName: "house"), tag: 0)
```

---

## 3. Свойства `UITabBarItem`

- **`title`** → текст вкладки
    
- **`image`** → иконка для обычного состояния
    
- **`selectedImage`** → иконка при активной вкладке
    
- **`badgeValue`** → строка для отображения бейджа
    
    ```swift
    homeVC.tabBarItem.badgeValue = "3" // Красный кружок с числом
    ```
    
- **`badgeColor`** (iOS 10+) → цвет бейджа
    

---

## 4. Кастомизация

Можно настраивать цвет текста и иконок через [[UITabBarAppearance]]:

```swift
let appearance = UITabBarAppearance()
appearance.stackedLayoutAppearance.normal.iconColor = .gray
appearance.stackedLayoutAppearance.selected.iconColor = .systemBlue
appearance.stackedLayoutAppearance.selected.titleTextAttributes = [.foregroundColor: UIColor.systemBlue]

UITabBar.appearance().standardAppearance = appearance
if #available(iOS 15.0, *) {
    UITabBar.appearance().scrollEdgeAppearance = appearance
}
```

---

## 5. Пример использования в `UITabBarController`

```swift
let homeVC = HomeViewController()
homeVC.tabBarItem = UITabBarItem(title: "Главная", image: UIImage(systemName: "house"), tag: 0)

let profileVC = ProfileViewController()
profileVC.tabBarItem = UITabBarItem(title: "Профиль", image: UIImage(systemName: "person"), tag: 1)

let tabBarController = UITabBarController()
tabBarController.viewControllers = [homeVC, profileVC]
```

---

## 6. Итог

- **`UITabBarItem`** = модель вкладки (`title + image + badge`).
    
- Привязывается к каждому `UIViewController` внутри `UITabBarController`.
    
- Можно легко кастомизировать (иконки, цвет текста, бейджи).
    

---
