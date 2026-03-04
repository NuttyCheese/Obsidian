#didFinishLaunchingWithOptions #appdelegate #scenedelegate #application-lifecycle #ios #swift #launch #initialization #rootviewcontroller #setup #ios-13

---
**(метод запуска приложения / точка входа в жизненный цикл)**

**application(_:didFinishLaunchingWithOptions:)** — это **самый первый** метод, который вызывается после запуска приложения (после splash-экрана), когда приложение переходит из состояния «not running» в «active» или «inactive».

Это **единственная точка**, где вы можете безопасно выполнить **одноразовую глобальную инициализацию** всего приложения.

С iOS 13 (2019) и введением **[[UIScene]]** метод немного изменил своё значение, но до сих пор остаётся крайне важным.

### 1. Где находится метод в 2026 году

Существует **два основных места**, где вы можете реализовать `didFinishLaunchingWithOptions`:

#### Вариант A. Классический [[AppDelegate]] (до iOS 13 и в простых приложениях)

```swift
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    var window: UIWindow?
    
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        // Здесь вся глобальная инициализация
        window = UIWindow(frame: UIScreen.main.bounds)
        window?.rootViewController = UINavigationController(rootViewController: HomeViewController())
        window?.makeKeyAndVisible()
        
        return true
    }
}
```

#### Вариант B. Современный [[SceneDelegate]] (iOS 13+ — рекомендуемый в 2026)

```swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    
    var window: UIWindow?
    
    func scene(_ scene: UIScene, willConnectTo session: UISceneSession,
               options connectionOptions: UIScene.ConnectionOptions) {
        
        guard let windowScene = (scene as? UIWindowScene) else { return }
        
        window = UIWindow(windowScene: windowScene)
        window?.rootViewController = UINavigationController(rootViewController: HomeViewController())
        window?.makeKeyAndVisible()
    }
}
```

**Важно в 2026:**  
- Если в Info.plist есть ключ `UIApplicationSceneManifest` → используется **SceneDelegate**  
- Если ключа нет → используется **AppDelegate** (старые проекты или минималистичные приложения)  
- Apple **настоятельно рекомендует** SceneDelegate для всех новых проектов

### 2. Что обычно делают в didFinishLaunchingWithOptions / scene(_:willConnectTo:)

| Задача                                      | Где лучше делать (2026)                          | Пример |
|---------------------------------------------|--------------------------------------------------|--------|
| Настройка глобального внешнего вида         | AppDelegate или SceneDelegate                    | `UINavigationBar.appearance()`, `UITabBar.appearance()` |
| Инициализация сторонних сервисов            | AppDelegate (один раз на запуск)                 | Firebase.configure(), Amplitude.setup(), Sentry.start |
| Регистрация уведомлений                     | AppDelegate                                      | `UNUserNotificationCenter.current().requestAuthorization` |
| Проверка авторизации / onboardings          | SceneDelegate (после создания окна)              | Показать Onboarding / Auth / MainTabBar |
| Установка rootViewController                | SceneDelegate                                    | `window?.rootViewController = ...` |
| Обработка launchOptions                     | AppDelegate или SceneDelegate                    | `launchOptions?[.url]` — открытие по ссылке |
| Настройка Coordinator                       | SceneDelegate                                    | `coordinator.start()` |
| Миграция данных / Core Data setup           | AppDelegate                                      | `persistentContainer.loadPersistentStores` |

### 3. Самый популярный и рекомендуемый паттерн 2026 года  
(SceneDelegate + [[Coordinator]] + Appearance + [[Firebase]] + Notifications)

```swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    
    var window: UIWindow?
    private var appCoordinator: AppCoordinator?
    
    func scene(_ scene: UIScene, willConnectTo session: UISceneSession,
               options connectionOptions: UIScene.ConnectionOptions) {
        
        guard let windowScene = (scene as? UIWindowScene) else { return }
        
        window = UIWindow(windowScene: windowScene)
        
        // 1. Глобальная настройка внешнего вида (один раз)
        setupGlobalAppearance()
        
        // 2. Инициализация сторонних сервисов
        FirebaseApp.configure()
        AnalyticsManager.setup()
        NotificationManager.registerForPushNotifications()
        
        // 3. Запуск координатора
        appCoordinator = AppCoordinator(window: window!)
        appCoordinator?.start()
        
        window?.makeKeyAndVisible()
    }
    
    private func setupGlobalAppearance() {
        let navAppearance = UINavigationBarAppearance()
        navAppearance.configureWithOpaqueBackground()
        navAppearance.backgroundColor = .systemBackground
        navAppearance.largeTitleTextAttributes = [.foregroundColor: UIColor.label]
        
        UINavigationBar.appearance().standardAppearance = navAppearance
        if #available(iOS 15.0, *) {
            UINavigationBar.appearance().scrollEdgeAppearance = navAppearance
        }
        
        let tabAppearance = UITabBarAppearance()
        tabAppearance.configureWithOpaqueBackground()
        tabAppearance.backgroundColor = .systemBackground
        
        UITabBar.appearance().standardAppearance = tabAppearance
        if #available(iOS 15.0, *) {
            UITabBar.appearance().scrollEdgeAppearance = tabAppearance
        }
    }
}
```

### 4. Лучшие практики didFinishLaunchingWithOptions / scene(_:willConnectTo:) в 2026 году

- **SceneDelegate** — предпочтительнее для новых проектов (iOS 13+)  
- **AppDelegate** — только для обратной совместимости или очень простых приложений  
- **Глобальная настройка** (appearance, сервисы) — делайте **один раз** здесь  
- **Coordinator** — для сложной навигации и создания контроллеров  
- **launchOptions** — проверяйте `.url`, `.remoteNotification`, `.shortcutItem` для deep linking и quick actions  
- **Не делайте** тяжёлых операций (сетевые запросы, миграцию БД) — делайте асинхронно или позже  
- **Документируйте** — пишите комментарий:

```swift
func scene(_ scene: UIScene, willConnectTo session: UISceneSession,
           options connectionOptions: UIScene.ConnectionOptions) {
    // Глобальная инициализация приложения
    setupAppearance()
    configureServices()
    startCoordinator()
}
```

**Короткий итог 2026**:
> **didFinishLaunchingWithOptions** / **scene(_:willConnectTo:)** — **первая точка запуска** приложения после splash-экрана.  
> В 2026 году:  
> - ключевые задачи — установка rootViewController, настройка appearance, инициализация сервисов, запуск координатора  
> - предпочтительнее **SceneDelegate** (iOS 13+)  
> - это **единственное место** для глобальной одноразовой настройки  
> - здесь решается, как быстро и красиво стартует ваше приложение  
