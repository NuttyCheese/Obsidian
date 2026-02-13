## 📘 Определение

**`SceneDelegate`** — это класс в [[iOS]], реализующий протокол **`UIWindowSceneDelegate`**, который отвечает за **жизненный цикл конкретной сцены приложения**.

Начиная с **iOS 13**, приложение может иметь **несколько сцен** (окна), особенно на iPad или при использовании многозадачности.  
`SceneDelegate` управляет:

- Созданием и отображением окна ([[UIWindow]]) сцены
    
- Переходом сцены между состояниями (активна, фон, отключена)
    
- Событиями жизненного цикла конкретного окна
    

Относится к: **UIKit → [[Application Life Cycle]] / Multi-Scene Support**

---

## 🔹 Основные методы `UIWindowSceneDelegate`

|Метод|Когда вызывается|Цель|
|---|---|---|
|`scene(_:willConnectTo:options:)`|При создании сцены|Настройка окна и rootViewController|
|`sceneDidDisconnect(_:)`|Сцена удалена системой|Освобождение ресурсов сцены|
|`sceneDidBecomeActive(_:)`|Сцена активна|Запуск анимаций, возобновление задач|
|`sceneWillResignActive(_:)`|Сцена временно неактивна|Приостановка работы интерфейса|
|`sceneWillEnterForeground(_:)`|Сцена выходит из фона|Подготовка интерфейса|
|`sceneDidEnterBackground(_:)`|Сцена ушла в фон|Сохранение данных, освобождение ресурсов|

---

## 🔹 Примеры кода

### 1. Стандартный `SceneDelegate`

```swift
import UIKit

class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?

    func scene(_ scene: UIScene,
               willConnectTo session: UISceneSession,
               options connectionOptions: UIScene.ConnectionOptions) {
        guard let windowScene = (scene as? UIWindowScene) else { return }

        window = UIWindow(windowScene: windowScene)
        window?.rootViewController = UIViewController()
        window?.makeKeyAndVisible()
    }
}
```

---

### 2. Обработка перехода сцены в активное состояние

```swift
func sceneDidBecomeActive(_ scene: UIScene) {
    print("Сцена активна")
}
```

---

### 3. Обработка ухода сцены в фон

```swift
func sceneDidEnterBackground(_ scene: UIScene) {
    print("Сцена ушла в фон")
}
```

---

### 4. Обработка отключения сцены

```swift
func sceneDidDisconnect(_ scene: UIScene) {
    print("Сцена отключена системой")
}
```

---

### 5. Настройка rootViewController с [[UINavigationController]]

```swift
func scene(_ scene: UIScene,
           willConnectTo session: UISceneSession,
           options connectionOptions: UIScene.ConnectionOptions) {

    guard let windowScene = (scene as? UIWindowScene) else { return }

    let navController = UINavigationController(rootViewController: UIViewController())
    window = UIWindow(windowScene: windowScene)
    window?.rootViewController = navController
    window?.makeKeyAndVisible()
}
```

---

### 🔹 Отличие от [[AppDelegate]]

|AppDelegate|SceneDelegate|
|---|---|
|Управляет **всем приложением**|Управляет **конкретной сценой** (окном)|
|До iOS 13 — единственный класс жизненного цикла|С iOS 13 — каждая сцена имеет отдельный SceneDelegate|
|Получает глобальные события (push, notification)|Получает события конкретной сцены (активная, фон)|
