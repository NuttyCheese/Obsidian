## application(_:didFinishLaunchingWithOptions:) — Точка входа в приложение

---

#ios #appdelegate #app-lifecycle #launch #swift

---

### Определение

**`application(_:didFinishLaunchingWithOptions:)`** — это первый метод, который вызывается в `AppDelegate` после запуска приложения. Он является **главной точкой входа** для выполнения любой инициализации, настройки сервисов и подготовки приложения к работе.

Этот метод вызывается **один раз** за время жизни приложения (если приложение не было убито системой и перезапущено). Он должен возвращать `true`, если приложение успешно запустилось.

```swift
func application(_ application: UIApplication,
                 didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    // Инициализация
    return true
}
```

---

### Зачем это знать iOS-разработчику?

| Сценарий                        | Почему это важно                                                        |
| ------------------------------- | ----------------------------------------------------------------------- |
| **Глобальная инициализация**    | Firebase, аналитика, Crashlytics, базы данных                           |
| **Настройка push-уведомлений**  | Регистрация на получение пушей                                          |
| **Обработка глубоких ссылок**   | Определение, как приложение было запущено (из URL, из push, из виджета) |
| **Первоначальная настройка UI** | Установка корневого контроллера                                         |
| **Проверка первого запуска**    | Онбординг, начальная настройка                                          |
| **Миграция данных**             | Обновление схемы [[Core Data]], миграция [[Realm]]                      |

---

### Параметры метода

| Параметр | Тип | Описание |
|---|---|---|
| **`application`** | `UIApplication` | Ссылка на объект приложения |
| **`launchOptions`** | `[UIApplication.LaunchOptionsKey: Any]?` | Словарь с информацией о причине запуска |

---

### Возможные причины запуска (LaunchOptionsKey)

| Ключ                                                   | Описание                                                               |
| ------------------------------------------------------ | ---------------------------------------------------------------------- |
| **`.url`**                                             | Приложение открыто через [[URL]]-схему (Universal Link, custom scheme) |
| **`.userActivity`**                                    | Приложение открыто через Handoff, Siri, Spotlight                      |
| **`.remoteNotification`**                              | Приложение открыто через push-уведомление                              |
| **`.localNotification`**                               | Приложение открыто через локальное уведомление                         |
| **`.shortcutItem`**                                    | Приложение открыто через 3D Touch / Haptic Touch меню                  |
| **`.location`**                                        | Приложение запущено из-за обновления геолокации                        |
| **`.bluetoothCentrals`** / **`.bluetoothPeripherals`** | Запуск из-за Bluetooth                                                 |

---

### Базовая структура AppDelegate

```swift
import UIKit
import Firebase
import UserNotifications

@main
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        print("🚀 didFinishLaunchingWithOptions — запуск приложения")
        
        // 1. Инициализация сервисов
        setupServices()
        
        // 2. Настройка UI
        setupAppearance()
        
        // 3. Настройка push
        setupPushNotifications(application)
        
        // 4. Обработка причины запуска
        handleLaunchOptions(launchOptions)
        
        // 5. Проверка первого запуска
        checkFirstLaunch()
        
        return true
    }
    
    // MARK: - Setup
    
    private func setupServices() {
        // Firebase
        FirebaseApp.configure()
        
        // Аналитика
        AnalyticsManager.shared.setup()
        
        // Crashlytics
        CrashlyticsManager.shared.setup()
        
        // База данных
        CoreDataManager.shared.setup()
    }
    
    private func setupAppearance() {
        // Настройка навигации
        UINavigationBar.appearance().tintColor = .systemBlue
        UINavigationBar.appearance().prefersLargeTitles = true
        
        // Настройка таб-бара
        UITabBar.appearance().tintColor = .systemBlue
    }
    
    private func setupPushNotifications(_ application: UIApplication) {
        UNUserNotificationCenter.current().delegate = self
        
        let authOptions: UNAuthorizationOptions = [.alert, .badge, .sound]
        UNUserNotificationCenter.current().requestAuthorization(options: authOptions) { granted, _ in
            print("Push authorization granted: \(granted)")
        }
        
        application.registerForRemoteNotifications()
    }
    
    private func handleLaunchOptions(_ options: [UIApplication.LaunchOptionsKey: Any]?) {
        guard let options = options else { return }
        
        // URL
        if let url = options[.url] as? URL {
            print("Launched with URL: \(url)")
            handleDeepLink(url)
        }
        
        // Push уведомление
        if let notification = options[.remoteNotification] as? [String: Any] {
            print("Launched with remote notification: \(notification)")
            handlePushNotification(notification)
        }
        
        // Shortcut (3D Touch)
        if let shortcut = options[.shortcutItem] as? UIApplicationShortcutItem {
            print("Launched with shortcut: \(shortcut.type)")
            handleShortcut(shortcut)
        }
        
        // User Activity (Handoff, Siri)
        if let userActivity = options[.userActivity] as? NSUserActivity {
            print("Launched with user activity: \(userActivity.activityType)")
            handleUserActivity(userActivity)
        }
    }
    
    private func checkFirstLaunch() {
        let isFirstLaunch = !UserDefaults.standard.bool(forKey: "hasLaunchedBefore")
        
        if isFirstLaunch {
            UserDefaults.standard.set(true, forKey: "hasLaunchedBefore")
            UserDefaults.standard.synchronize()
            
            // Первый запуск
            showOnboarding()
            preloadInitialData()
        }
    }
    
    // MARK: - Handlers
    private func handleDeepLink(_ url: URL) { }
    private func handlePushNotification(_ userInfo: [String: Any]) { }
    private func handleShortcut(_ shortcut: UIApplicationShortcutItem) { }
    private func handleUserActivity(_ activity: NSUserActivity) { }
    private func showOnboarding() { }
    private func preloadInitialData() { }
}
```

---

### Обработка разных сценариев запуска

#### 1. Запуск через URL Scheme

```swift
// В Info.plist нужно добавить URL Scheme
// myapp://product/123

func application(_ app: UIApplication,
                 open url: URL,
                 options: [UIApplication.OpenURLOptionsKey: Any] = [:]) -> Bool {
    
    print("Open URL: \(url)")
    
    if url.scheme == "myapp" {
        // Парсинг URL
        let components = url.pathComponents
        if components.contains("product"), let id = components.last {
            navigateToProduct(withId: id)
        }
    }
    
    return true
}
```

#### 2. Запуск через Shortcut (3D Touch)

```swift
// В Info.plist нужно добавить UIApplicationShortcutItems
func application(_ application: UIApplication,
                 performActionFor shortcutItem: UIApplicationShortcutItem,
                 completionHandler: @escaping (Bool) -> Void) {
    
    switch shortcutItem.type {
    case "com.app.search":
        navigateToSearch()
        completionHandler(true)
    case "com.app.profile":
        navigateToProfile()
        completionHandler(true)
    default:
        completionHandler(false)
    }
}
```

#### 3. Запуск через Universal Link

```swift
// В Associated Domains нужно добавить applinks:example.com
func application(_ application: UIApplication,
                 continue userActivity: NSUserActivity,
                 restorationHandler: @escaping ([UIUserActivityRestoring]?) -> Void) -> Bool {
    
    guard userActivity.activityType == NSUserActivityTypeBrowsingWeb,
          let url = userActivity.webpageURL else {
        return false
    }
    
    handleDeepLink(url)
    return true
}
```

#### 4. Запуск через Push Notification

```swift
func application(_ application: UIApplication,
                 didReceiveRemoteNotification userInfo: [AnyHashable: Any],
                 fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {
    
    // Приложение активно или в фоне
    if application.applicationState == .active {
        // Приложение активно — показать уведомление
        showInAppNotification(userInfo)
    } else {
        // Приложение запустилось из пуша
        handlePushNotification(userInfo)
    }
    
    completionHandler(.newData)
}
```

---

### SceneDelegate (iOS 13+)

Начиная с [[iOS]] 13, для приложений с поддержкой сцен, `didFinishLaunchingWithOptions` используется только для глобальной инициализации, а создание UI переехало в `SceneDelegate`.

```swift
@main
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        // Только глобальная инициализация
        FirebaseApp.configure()
        
        return true
    }
    
    // MARK: - UISceneSession Lifecycle
    func application(_ application: UIApplication,
                     configurationForConnecting connectingSceneSession: UISceneSession,
                     options: UIScene.ConnectionOptions) -> UISceneConfiguration {
        
        return UISceneConfiguration(name: "Default Configuration", sessionRole: connectingSceneSession.role)
    }
}

class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    
    var window: UIWindow?
    
    func scene(_ scene: UIScene,
               willConnectTo session: UISceneSession,
               options connectionOptions: UIScene.ConnectionOptions) {
        
        guard let windowScene = scene as? UIWindowScene else { return }
        
        window = UIWindow(windowScene: windowScene)
        window?.rootViewController = MainViewController()
        window?.makeKeyAndVisible()
        
        // Обработка причины запуска для сцены
        if let url = connectionOptions.urlContexts.first?.url {
            handleDeepLink(url)
        }
        
        if let shortcut = connectionOptions.shortcutItem {
            handleShortcut(shortcut)
        }
        
        if let userActivity = connectionOptions.userActivities.first {
            handleUserActivity(userActivity)
        }
    }
}
```

---

### Производительность и оптимизация

| Рекомендация | Почему |
|---|---|
| **Не делайте тяжёлых операций синхронно** | Задерживает запуск, пользователь видит чёрный экран |
| **Выносите долгие задачи в фоновые очереди** | Приложение запускается быстрее |
| **Используйте асинхронную инициализацию** | Firebase и другие сервисы поддерживают async init |
| **Кэшируйте результаты инициализации** | При повторных запусках будет быстрее |

```swift
// ✅ Хорошо: асинхронная инициализация
func application(_ application: UIApplication,
                 didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    
    // Быстрые инициализации
    setupCrashlytics()
    setupAppearance()
    
    // Долгие операции — асинхронно
    DispatchQueue.global(qos: .background).async {
        self.migrateDatabase()
        self.preloadLargeData()
    }
    
    return true
}
```

---

### Типичные ошибки

#### 1. Возврат false

```swift
// ❌ Неправильно — приложение не запустится
func application(...) -> Bool {
    return false
}

// ✅ Правильно — всегда возвращайте true
func application(...) -> Bool {
    // Инициализация...
    return true
}
```

#### 2. Долгая синхронная инициализация

```swift
// ❌ Плохо — задержка запуска
func application(...) -> Bool {
    migrateLargeDatabase()  // Синхронно, может занять секунды
    return true
}
```

#### 3. Игнорирование launchOptions

```swift
// ❌ Плохо — теряем информацию о том, почему запустилось приложение
func application(...) -> Bool {
    setupServices()
    return true
}

// ✅ Хорошо — обрабатываем причину запуска
func application(...) -> Bool {
    setupServices()
    
    if let launchOptions = launchOptions {
        // Обработка
    }
    
    return true
}
```

#### 4. Duplicate initialization

```swift
// ❌ Плохо — повторная инициализация при повторных запусках
func application(...) -> Bool {
    setupServices()  // Вызывается при каждом запуске
    return true
}

// ✅ Хорошо — проверяем, инициализированы ли уже сервисы
private var didSetupServices = false

func application(...) -> Bool {
    if !didSetupServices {
        setupServices()
        didSetupServices = true
    }
    return true
}
```

---

### Лучшие практики (2026)

| Практика | Почему |
|---|---|
| **Инициализируйте только глобальные сервисы** | Firebase, аналитика, Crashlytics, базы данных |
| **Не создавайте UI в `didFinishLaunching`** | Используйте SceneDelegate для iOS 13+ |
| **Обрабатывайте `launchOptions`** | Для глубоких ссылок, пушей, shortcut |
| **Делайте инициализацию асинхронной** | Ускоряет запуск |
| **Проверяйте первый запуск** | Для онбординга и начальной настройки |
| **Возвращайте `true` всегда** | Иначе приложение не запустится |

---

### Короткое правило

> **`didFinishLaunchingWithOptions`** — первая и единственная точка входа для глобальной инициализации.  
> **Не создавай здесь UI** (для этого есть SceneDelegate).  
> **Обрабатывай `launchOptions`**, чтобы понять, почему приложение запустилось.  
> **Делай инициализацию быстрой**, тяжёлые задачи — в фон.  
> **Всегда возвращай `true`**.

---

### Итог

**`application(_:didFinishLaunchingWithOptions:)`** — самый важный метод в жизненном цикле приложения:

| Аспект | Значение |
|---|---|
| **Вызывается** | 1 раз при запуске |
| **Назначение** | Глобальная инициализация сервисов |
| **Возврат** | `true` — успех, `false` — приложение не запустится |
| **UI** | Не создавать (используй SceneDelegate) |
| **Тяжёлые операции** | Асинхронно (в фоновой очереди) |
| **`launchOptions`** | Всегда обрабатывай (глубокие ссылки, пуши) |

**Главное правило:**
> Всегда возвращай `true`. Инициализируй только глобальные сервисы. Не создавай UI. Тяжёлые операции выноси в фон. Обрабатывай `launchOptions`, чтобы правильно реагировать на запуск через URL, push или shortcut. Для iOS 13+ используй SceneDelegate для UI-логики. Проверяй первый запуск для показа онбординга. Не делай синхронных долгих операций — это задерживает запуск и портит пользовательский опыт.