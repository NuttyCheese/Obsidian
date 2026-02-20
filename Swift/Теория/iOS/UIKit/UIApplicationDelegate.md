**UIApplicationDelegate** — это **протокол** в [[UIKit]], который определяет методы, связанные с **жизненным циклом всего приложения** (не отдельного контроллера, а именно приложения в целом).

С 2019–2020 годов ([[iOS]] 13+) его роль сильно уменьшилась из-за появления **[[SceneDelegate]]** и **[[UIScene]]**-системы, но в 2026 году он **по-прежнему важен** и активно используется в большинстве приложений.

### Ключевые моменты 2026 года

| Ситуация / Тип приложения                                                     | Нужно ли реализовывать UIApplicationDelegate? | Где обычно живут методы жизненного цикла |
| ----------------------------------------------------------------------------- | --------------------------------------------- | ---------------------------------------- |
| Приложение **без сцен** (iOS 12 и ниже, или legacy)                           | **Да**, все методы здесь                      | [[AppDelegate]]                          |
| Приложение **с сценами** (iOS 13+), но **одно окно** (большинство приложений) | **Да**, но минимально                         | AppDelegate + SceneDelegate              |
| Приложение **с несколькими сценами** (multi-window, iPadOS, внешний дисплей)  | **Да**, но почти пустой                       | SceneDelegate берёт основную работу      |
| SwiftUI-приложение (с @main App)                                              | **Нет** (или минимально)                      | @main App + AppDelegate (опционально)    |

### Основные методы UIApplicationDelegate (актуальные в 2026)

| Метод                                                | Когда вызывается (основные случаи)                                                   | Что обычно делают внутри (2026)                                                                | Обязателен?  |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------- | ------------ |
| `application(_:didFinishLaunchingWithOptions:)`      | При запуске приложения (самый важный метод)                                          | Настройка AppDelegate, делегатов, [[Firebase]], аналитики, разрешения, начальная маршрутизация | Почти всегда |
| `applicationWillResignActive(_:)`                    | Приложение теряет активность (входящий звонок, Control Center, переход в background) | Приостановка анимаций, музыки, игр, сохранение состояния                                       | Редко        |
| `applicationDidEnterBackground(_:)`                  | Приложение ушло в фон                                                                | Сохранение состояния, остановка таймеров, запрос background tasks                              | Часто        |
| `applicationWillEnterForeground(_:)`                 | Приложение возвращается на передний план                                             | Обновление UI, возобновление таймеров, проверка авторизации                                    | Часто        |
| `applicationDidBecomeActive(_:)`                     | Приложение полностью активно (после foreground)                                      | Запуск анимаций, синхронизация, возобновление сетевых запросов                                 | Часто        |
| `applicationWillTerminate(_:)`                       | Приложение завершается (редко вызывается)                                            | Финальное сохранение данных (очень ненадёжно)                                                  | Редко        |
| `application(_:configurationForConnecting:options:)` | Создание новой сцены (iOS 13+)                                                       | Возврат конфигурации сцены (SceneDelegate)                                                     | При сценах   |
| `application(_:didDiscardSceneSessions:)`            | Пользователь закрыл сцены в App Switcher                                             | Очистка ресурсов для отброшенных сцен                                                          | При сценах   |

### Современный шаблон AppDelegate в 2026 году (минимальный + сцены)

```swift
import UIKit

@main
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        // Здесь только глобальная инициализация, которая нужна до первой сцены
        setupGlobalDependencies()
        requestPermissionsIfNeeded()
        configureThirdPartySDKs()
        
        return true
    }
    
    // MARK: UISceneSession Lifecycle
    
    func application(_ application: UIApplication,
                     configurationForConnecting connectingSceneSession: UISceneSession,
                     options: UIScene.ConnectionOptions) -> UISceneConfiguration {
        
        // Создаём конфигурацию для новой сцены
        let sceneConfig = UISceneConfiguration(name: "Default Configuration", sessionRole: connectingSceneSession.role)
        sceneConfig.delegateClass = SceneDelegate.self
        
        return sceneConfig
    }
    
    func application(_ application: UIApplication,
                     didDiscardSceneSessions sceneSessions: Set<UISceneSession>) {
        // Очистка ресурсов для закрытых сцен (редко нужно)
    }
}
```

### Современный шаблон SceneDelegate (основная работа в 2026)

```swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    
    var window: UIWindow?
    
    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        
        guard let windowScene = (scene as? UIWindowScene) else { return }
        
        let window = UIWindow(windowScene: windowScene)
        self.window = window
        
        // Настраиваем корневой контроллер
        let rootVC = MainTabBarController()
        window.rootViewController = rootVC
        window.makeKeyAndVisible()
        
        // Обработка запуска по deep link / уведомлению
        if let shortcut = connectionOptions.shortcutItem {
            handleShortcut(shortcut)
        }
    }
    
    func sceneDidBecomeActive(_ scene: UIScene) {
        // Приложение активно → можно запускать анимации, синхронизацию
    }
    
    func sceneWillResignActive(_ scene: UIScene) {
        // Приложение теряет активность → пауза
    }
    
    func sceneDidEnterBackground(_ scene: UIScene) {
        // Сохранение состояния
    }
    
    func sceneWillEnterForeground(_ scene: UIScene) {
        // Возобновление
    }
}
```

### Лучшие практики UIApplicationDelegate в 2026 году

- **Минимизируйте** код в **AppDelegate** — только глобальная инициализация (Firebase, аналитика, Crashlytics, разрешения)  
- **Основную работу** (настройка UI, обработка deep link, уведомлений) делайте в **SceneDelegate**  
- **Для SwiftUI** — используйте `@main struct App` + `AppDelegate` (опционально) через `@UIApplicationDelegateAdaptor`  
- **Не полагайтесь** на `applicationWillTerminate` — он почти никогда не вызывается (система убивает приложение без предупреждения)  
- **Для multi-window** (iPadOS) — вся логика создания окон должна быть в `configurationForConnecting`  
- **Документируйте** — пишите комментарий «application(_:didFinishLaunchingWithOptions:) — глобальная инициализация приложения (Firebase, аналитика, разрешения)»

**Короткий итог 2026**:
> UIApplicationDelegate — это **протокол для управления жизненным циклом всего приложения**.  
> В 2026 году:  
> - главный метод — `didFinishLaunchingWithOptions` (глобальная инициализация)  
> - при iOS 13+ основная работа переехала в **SceneDelegate**  
> - AppDelegate стал **тонким** (только глобальные сервисы)  
> - для SwiftUI — используйте `@UIApplicationDelegateAdaptor`  
> Это **классический**, но уже **не главный** игрок в жизненном цикле приложения.
