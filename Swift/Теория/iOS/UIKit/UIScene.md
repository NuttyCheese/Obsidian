**UIScene** — это фундаментальный класс в iOS 13+ (2019), который представляет **одну независимую сцену** (scene) приложения — по сути, **одно окно** или **один экземпляр интерфейса** приложения.

С появлением `UIScene` Apple перешла от модели «одно окно на всё приложение» к модели **множественных сцен** (multiple scenes), что позволило:

- запускать несколько окон одного приложения одновременно (iPad, Mac Catalyst)
- поддерживать Split View, Slide Over, Stage Manager
- подключать внешние дисплеи
- восстанавливать состояние отдельных окон отдельно

### Ключевые моменты о UIScene (актуально на 2026 год)

| Характеристика                       | Описание                                                              | Почему важно в 2026                                             |
| ------------------------------------ | --------------------------------------------------------------------- | --------------------------------------------------------------- |
| Представляет одно окно/сцену         | Каждое окно = отдельная `UIScene` (обычно [[UIWindowScene]])          | Поддержка многоконности                                         |
| Жизненный цикл управляется делегатом | [[UISceneDelegate]] — основной делегат для каждой сцены               | `willConnectTo`, `didBecomeActive`, `didEnterBackground` и т.д. |
| Связана с `UISceneSession`           | `UISceneSession` хранит конфигурацию и состояние для восстановления   | Восстановление после kill приложения                            |
| Может быть нескольких типов          | `UIWindowScene`, `UIWindowSceneSession.Role.windowApplication` и т.д. | Разные роли: окно, внешний дисплей                              |
| Поддерживает внешние дисплеи         | С iPadOS 13+ и macOS Catalyst можно выводить контент на монитор       | Приложения с расширенным экраном                                |
| Связана с `UIWindow`                 | Каждая сцена обычно имеет одно основное окно (`keyWindow`)            | Доступ к rootViewController                                     |

### Основные подклассы UIScene

| Класс                              | Когда используется                                       | Самый частый сценарий в 2026 |
|------------------------------------|----------------------------------------------------------|------------------------------|
| `UIWindowScene`                    | Стандартное окно приложения                              | Почти все сцены — это `UIWindowScene` |
| `UIWindowScene` + внешний дисплей  | Подключение HDMI/USB-C/ AirPlay                          | Расширенный экран на iPad/Mac |
| `UIScene` (абстрактный)            | Базовый класс, редко используется напрямую              | — |

### Самый популярный и рекомендуемый паттерн 2026 года  
([[SceneDelegate]] + [[UIWindowScene]] + [[SwiftUI]]/[[UIKit]])

```swift
import UIKit

class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    
    var window: UIWindow?
    
    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        guard let windowScene = (scene as? UIWindowScene) else { return }
        
        let window = UIWindow(windowScene: windowScene)
        self.window = window
        
        // Самый частый выбор 2026 — SwiftUI root
        if #available(iOS 14.0, *) {
            window.rootViewController = UIHostingController(rootView: ContentView())
        } else {
            // UIKit fallback
            let nav = UINavigationController(rootViewController: MainViewController())
            window.rootViewController = nav
        }
        
        window.makeKeyAndVisible()
        
        // Обработка deep link при запуске
        if let userActivity = connectionOptions.userActivities.first {
            scene(scene, continue: userActivity)
        }
        
        // Проверка ориентации и size class
        print("Новая сцена создана, ориентация:", windowScene.interfaceOrientation)
    }
    
    func sceneDidBecomeActive(_ scene: UIScene) {
        // Сцена на экране — можно запускать обновления данных
        print("Сцена стала активной")
    }
    
    func sceneWillResignActive(_ scene: UIScene) {
        // Сохраняем состояние перед уходом в фон
        saveSceneState(for: scene)
    }
    
    func sceneDidEnterBackground(_ scene: UIScene) {
        // Экономим ресурсы
    }
    
    func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
        // Обработка custom URL scheme
        guard let url = URLContexts.first?.url else { return }
        handleCustomURL(url)
    }
    
    private func handleCustomURL(_ url: URL) {
        // Пример: myapp://profile/123 → открыть профиль
    }
    
    private func saveSceneState(for scene: UIScene) {
        // Сохранение в UserDefaults или Keychain
    }
}
```

### Получение текущей сцены (самый частый код в контроллерах)

```swift
extension UIViewController {
    var currentScene: UIScene? {
        view.window?.windowScene
    }
    
    var currentWindowScene: UIWindowScene? {
        currentScene as? UIWindowScene
    }
    
    var currentInterfaceOrientation: UIInterfaceOrientation? {
        currentWindowScene?.interfaceOrientation
    }
}

// Использование
if let scene = currentWindowScene {
    print("Текущая ориентация сцены:", scene.interfaceOrientation)
}
```

### Лучшие практики UIScene в 2026 году

- **Всегда** работайте через `UIWindowScene` — это основной подкласс `UIScene`  
- **Для доступа к окну** — используйте `windowScene?.keyWindow` (iOS 15+) вместо устаревшего `UIApplication.shared.keyWindow`  
- **Для нескольких сцен** — перебирайте `UIApplication.shared.connectedScenes` и фильтруйте по `UIWindowScene`  
- **Для внешних дисплеев** — слушайте `UIScreen.didConnectNotification` и создавайте новую сцену  
- **Для SwiftUI** — `UIHostingController` в `scene(_:willConnectTo:)` — `UISceneDelegate` нужен только в UIKit или смешанных проектах  
- **Для восстановления состояния** — используйте `NSUserActivity` + `stateRestorationActivity(for:)`  
- **Документируйте** — пишите комментарий:

```swift
/// Доступ к текущей UIWindowScene из любого контроллера
var currentWindowScene: UIWindowScene? {
    view.window?.windowScene
}
```

**Короткий итог 2026**:
> `UIScene` — это **абстракция одной сцены/окна** в iOS 13+ архитектуре.  
> В 2026 году:  
> - почти всегда используется как `UIWindowScene`  
> - управляется через `UISceneDelegate`  
> - ключевой метод — `scene(_:willConnectTo:options:)` (установка root VC)  
> - важно для многоконности, Split View, Stage Manager, внешних дисплеев  
> - это **основа** современной многозадачности в UIKit-приложениях  
