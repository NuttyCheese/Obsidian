## 📘 Определение

**`AppDelegate`** — это класс в [[iOS]], реализующий протокол **`UIApplicationDelegate`**, который отвечает за **жизненный цикл приложения** и глобальные события, такие как запуск, переход в фон, получение уведомлений и т.д.  
Относится к **[[UIKit]] → [[Application Life Cycle]]**.

---

## 🔹 Примеры кода

### 1. Стандартный `AppDelegate`

```swift
import UIKit

@main
class AppDelegate: UIResponder, UIApplicationDelegate {

    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        print("Приложение запущено")
        return true
    }
}
```

---

### 2. Обработка перехода в фон

```swift
func applicationDidEnterBackground(_ application: UIApplication) {
    print("Приложение ушло в фон")
}
```

---

### 3. Обработка перехода в активное состояние

```swift
func applicationDidBecomeActive(_ application: UIApplication) {
    print("Приложение стало активным")
}
```

---

### 4. Обработка завершения приложения

```swift
func applicationWillTerminate(_ application: UIApplication) {
    print("Приложение завершает работу")
}
```

---

### 5. Получение уведомлений

```swift
func application(_ application: UIApplication,
                 didReceiveRemoteNotification userInfo: [AnyHashable : Any],
                 fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {
    print("Получено push-уведомление:", userInfo)
    completionHandler(.newData)
}
```

---

### 6. Настройка глобального окна (для [[UIKit]] без [[SceneDelegate]])

```swift
var window: UIWindow?

func application(_ application: UIApplication,
                 didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    window = UIWindow(frame: UIScreen.main.bounds)
    window?.rootViewController = UIViewController()
    window?.makeKeyAndVisible()
    return true
}
```
