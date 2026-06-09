**UISceneDelegate** — это протокол в [[UIKit]], который управляет **жизненным циклом сцены** (scene) в iOS-приложениях, начиная с iOS 13 (2019 год).

Он пришёл на смену старому [[UIApplicationDelegate]] для обработки событий конкретной сцены (окна), а не всего приложения.  
Теперь каждое окно (сцена) может иметь **свой собственный делегат**, что критично для поддержки **многоконности** (multiple scenes) на iPad, Mac Catalyst и внешних дисплеях.

### Когда UISceneDelegate нужен в 2026 году

| Сценарий                                      | Почему именно UISceneDelegate                                | Альтернатива |
|-----------------------------------------------|---------------------------------------------------------------|--------------|
| Поддержка нескольких окон (Split View, Slide Over, Stage Manager) | Каждая сцена имеет свой делегат                               | — |
| Отдельное поведение при открытии/закрытии окна | Разные rootViewController, разные navigation stacks           | — |
| Обработка восстановления состояния сцены      | `scene(_:willConnectTo:options:)` + `userActivity`            | `NSUserActivity` |
| Работа с внешними дисплеями                   | `UIScreen.didConnectNotification` → создание новой сцены      | — |
| Кастомный lifecycle сцены (foreground/background) | События сцены, а не всего приложения                         | `UIApplicationDelegate` (только для app-level) |

### Основные методы протокола UISceneDelegate (самые важные)

| Метод делегата                    | Когда вызывается                                                | Что обычно делают внутри метода                                  | Самый частый сценарий                |
| --------------------------------- | --------------------------------------------------------------- | ---------------------------------------------------------------- | ------------------------------------ |
| [[scene(willConnectTo options)]]  | Когда сцена создаётся или восстанавливается (самый важный!)     | Установка rootViewController, загрузка состояния из userActivity | Установка начального экрана          |
| [[sceneDidBecomeActive]]          | Сцена стала активной (на экране)                                | Запуск анимаций, обновление UI, возобновление сетевых запросов   | Начало работы сцены                  |
| [[sceneWillResignActive]]         | Сцена теряет активность (например, переключение на другое окно) | Сохранение состояния, пауза анимаций, остановка таймеров         | Подготовка к уходу в фон             |
| [[sceneDidEnterBackground]]       | Сцена ушла в фон                                                | Сохранение данных, отключение ненужных обновлений                | Экономия батареи                     |
| [[sceneWillEnterForeground]]      | Сцена возвращается на экран                                     | Обновление данных, перезапуск анимаций                           | Возобновление работы                 |
| [[sceneDidDisconnect]]            | Сцена уничтожается (окно закрыто)                               | Очистка ресурсов, отмена подписок                                | Завершение сцены                     |
| [[scene(openURLContexts)]]        | Открытие по [[URL]] ([[universal link]]s, custom schemes)       | Обработка [[deep link]]                                          | Deep linking                         |
| [[stateRestorationActivity(for)]] | Сохранение состояния сцены для восстановления                   | Создание NSUserActivity с данными                                | Восстановление после kill приложения |
| [[scene(continue)]]               | Восстановление из userActivity                                  | Восстановление состояния из NSUserActivity                       | Handoff, universal links             |

### Самый популярный и рекомендуемый паттерн 2026 года  
([[SceneDelegate]] + [[SwiftUI]]/[[UIKit]] + [[Swift Concurrency]])

```swift
import UIKit

class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    
    var window: UIWindow?
    
    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        guard let windowScene = (scene as? UIWindowScene) else { return }
        
        let window = UIWindow(windowScene: windowScene)
        self.window = window
        
        // Самый частый выбор 2026 — SwiftUI + UIKit
        if #available(iOS 14.0, *) {
            // SwiftUI root
            window.rootViewController = UIHostingController(rootView: ContentView())
        } else {
            // UIKit root
            let navController = UINavigationController(rootViewController: MainViewController())
            window.rootViewController = navController
        }
        
        window.makeKeyAndVisible()
        
        // Обработка deep link / universal link при запуске
        if let userActivity = connectionOptions.userActivities.first {
            scene(scene, continue: userActivity)
        }
    }
    
    func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
        // Обработка universal link / Handoff
        if userActivity.activityType == NSUserActivityTypeBrowsingWeb,
           let incomingURL = userActivity.webpageURL {
            handleDeepLink(url: incomingURL)
        }
    }
    
    func sceneDidBecomeActive(_ scene: UIScene) {
        // Сцена на экране — можно запускать обновления
        print("Сцена стала активной")
    }
    
    func sceneDidEnterBackground(_ scene: UIScene) {
        // Сохраняем состояние
        saveAppState()
    }
    
    private func handleDeepLink(url: URL) {
        // Пример: открыть профиль пользователя
        if url.pathComponents.contains("profile"),
           let userId = url.lastPathComponent {
            // переход на экран профиля
        }
    }
    
    private func saveAppState() {
        // Сохранение в UserDefaults / Keychain / Core Data
    }
}
```

### Лучшие практики UISceneDelegate в 2026 году

- **Всегда** реализуйте `scene(_:willConnectTo:options:)` — это точка входа для каждой сцены  
- **Используйте** `windowScene?.keyWindow?.rootViewController` для доступа к root VC текущей сцены  
- **Для deep linking** — обрабатывайте в `scene(_:continue:)` и `application(_:continue:restorationHandler:)`  
- **Для многоконности** — проверяйте `UIApplication.shared.connectedScenes` и фильтруйте по `UIWindowScene`  
- **Для SwiftUI** — `UIHostingController` в `scene(_:willConnectTo:)` — SceneDelegate нужен только в UIKit или смешанных проектах  
- **Для восстановления состояния** — используйте `NSUserActivity` + `stateRestorationActivity(for:)`  
- **Документируйте** — пишите комментарий:

```swift
/// SceneDelegate управляет жизненным циклом каждой сцены (окна) приложения
class SceneDelegate: UIResponder, UIWindowSceneDelegate { ... }
```

**Короткий итог 2026**:
> `UISceneDelegate` — протокол для управления **жизненным циклом отдельной сцены** (окна) в iOS 13+.  
> В 2026 году:  
> - ключевой метод — `scene(_:willConnectTo:options:)` (установка root VC)  
> - важен для многоконности, Split View, Stage Manager, внешних дисплеев  
> - основные события — `didBecomeActive`, `didEnterBackground`, `openURLContexts`  
> - это **основа** современной архитектуры UIKit-приложений с несколькими окнами  
