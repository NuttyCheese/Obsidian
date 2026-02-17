**`UITabBar`** — это **панель вкладок** внизу экрана, которая позволяет переключаться между различными разделами приложения.  
Она является частью **[[UITabBarController]]**, но как самостоятельный элемент может использоваться для кастомных интерфейсов.

🔑 Основное:

- Обычно располагается **внизу экрана**.
    
- Состоит из **элементов ([[UITabBarItem]])** — кнопок с иконками и/или текстом.
    
- Позволяет пользователю быстро переключаться между разделами приложения.
    

---

## 2. Где используется

Чаще всего `UITabBar` используется внутри **`UITabBarController`**:

- Каждая вкладка = отдельный [[UIViewController]] или [[UINavigationController]].
    
- Пользователь видит **до 5 вкладок одновременно**. Если вкладок больше → появляется вкладка **"More"**.
    

Пример из реальных приложений:

- Instagram (Главная, Поиск, Рилсы, Магазин, Профиль)
    
- Safari (Закладки, История, Открытые вкладки, и т.д.)
    

---

## 3. Пример кода

### Создание таббара через `UITabBarController`:

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

👉 В этом примере создаются 2 вкладки: "Главная" и "Профиль".

---

## 4. Настройка `UITabBar`

Через `UITabBarAppearance` можно кастомизировать цвет, фон, шрифт:

```swift
let appearance = UITabBarAppearance()
appearance.backgroundColor = .systemGray6
appearance.stackedLayoutAppearance.selected.iconColor = .systemBlue
appearance.stackedLayoutAppearance.selected.titleTextAttributes = [.foregroundColor: UIColor.systemBlue]

UITabBar.appearance().standardAppearance = appearance
if #available(iOS 15.0, *) {
    UITabBar.appearance().scrollEdgeAppearance = appearance
}
```

---

## 5. Важные свойства `UITabBarItem`

- `title` — подпись под иконкой
    
- `image` — иконка обычного состояния
    
- `selectedImage` — иконка при выборе
    
- `badgeValue` — маленький красный "бейджик" (например, количество уведомлений)
    

Пример:

```swift
let item = UITabBarItem(title: "Сообщения", image: UIImage(systemName: "message"), tag: 0)
item.badgeValue = "5"
```

---

## 6. Отличие `UITabBar` и `UITabBarController`

- **`UITabBar`** — только сама панель (UI-элемент).
    
- **`UITabBarController`** — контроллер, который управляет `UITabBar` и связанными `UIViewController`.
    

Обычно мы напрямую работаем с **`UITabBarController`**, а не с `UITabBar`.

---

## 7. Итог

- `UITabBar` = нижняя панель навигации.
    
- Работает через **`UITabBarController`**.
    
- Поддерживает кастомизацию, иконки, бейджи.
    
- Лучший выбор для приложений, где **несколько независимых разделов**.
    

---
