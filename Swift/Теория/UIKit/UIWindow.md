**`UIWindow`** — это **базовый контейнер для всех визуальных элементов приложения** в [[iOS]].  
Представляет собой **главное окно приложения**, которое отображает **[[Swift/Теория/UIKit/UIView]] и [[UIViewController]]**.  
Каждое iOS-приложение имеет как минимум одно окно.  
Относится к **UIKit → App Life Cycle / UI Components**.

---

## 🔹 Примеры кода

### 1. Создание UIWindow вручную

```swift
import UIKit

let window = UIWindow(frame: UIScreen.main.bounds)
window.backgroundColor = .white
window.makeKeyAndVisible()
```

---

### 2. Назначение корневого контроллера

```swift
let viewController = UIViewController()
viewController.view.backgroundColor = .systemBlue

window.rootViewController = viewController
window.makeKeyAndVisible()
```

---

### 3. Использование [[SceneDelegate]] (iOS 13+)

```swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    var window: UIWindow?

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession,
               options connectionOptions: UIScene.ConnectionOptions) {
        guard let windowScene = (scene as? UIWindowScene) else { return }
        window = UIWindow(windowScene: windowScene)
        window?.rootViewController = ViewController()
        window?.makeKeyAndVisible()
    }
}
```

---

### 4. Проверка ключевого окна

```swift
if let keyWindow = UIApplication.shared.keyWindow {
    print("Key window:", keyWindow)
}
```

---

### 5. Добавление overlay view

```swift
let overlay = UIView(frame: window.bounds)
overlay.backgroundColor = UIColor.black.withAlphaComponent(0.5)
window.addSubview(overlay)
```
