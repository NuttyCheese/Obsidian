**AppDelegate** — это центральный класс в UIKit-приложениях, который реализует протокол **`UIApplicationDelegate`**.

Он отвечает за **жизненный цикл всего приложения** и получает уведомления о ключевых системных событиях.

### Когда и зачем нужен AppDelegate в 2026 году

| Сценарий                                      | Почему именно AppDelegate                              | Альтернатива в современных проектах |
|-----------------------------------------------|--------------------------------------------------------|-------------------------------------|
| Настройка при запуске приложения              | `didFinishLaunchingWithOptions` — первое место, где можно что-то сделать | SceneDelegate / @main + App struct (SwiftUI) |
| Обработка push-уведомлений (remote/local)     | `didReceiveRemoteNotification`, `didRegisterForRemoteNotifications` | UNUserNotificationCenterDelegate |
| Управление состоянием приложения (фон/активное) | `applicationDidEnterBackground`, `applicationDidBecomeActive` | SceneDelegate (в сценах) |
| Настройка глобального окна (legacy UIKit)     | Создание `window` и `rootViewController`               | UIScene / UIWindowScene в iOS 13+ |
| Регистрация сторонних сервисов (Firebase, Amplitude, Crashlytics) | Один раз при запуске                                   | AppDelegate или @main init |
| Обработка URL-схем / Universal Links          | `application(_:open:options:)`                         | SceneDelegate или SwiftUI onOpenURL |
| Управление ориентацией / поддержкой multitasking | `supportedInterfaceOrientationsFor`                    | Info.plist + SceneDelegate |
| Завершение приложения / сохранение состояния   | `applicationWillTerminate` (редко вызывается)          | `applicationDidEnterBackground` |

### Современный AppDelegate 2026 (минимальный + полезный)

```swift
import UIKit

@main
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    // 1. Самый важный метод — запуск приложения
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        // Настройка сторонних сервисов (один раз при запуске)
        FirebaseApp.configure()
        Amplitude.instance().initialize(apiKey: "...")
        Crashlytics.crashlytics().setCrashlyticsCollectionEnabled(true)
        
        // Регистрация на push-уведомления
        UNUserNotificationCenter.current().delegate = self
        application.registerForRemoteNotifications()
        
        // Логирование запуска
        print("🚀 Приложение запущено. LaunchOptions: \(launchOptions ?? [:])")
        
        return true
    }
    
    // 2. Приложение стало активным (возврат из фона, первый запуск)
    func applicationDidBecomeActive(_ application: UIApplication) {
        print("✅ Приложение стало активным")
        
        // Можно обновить виджеты, проверить токены и т.д.
    }
    
    // 3. Приложение ушло в фон
    func applicationDidEnterBackground(_ application: UIApplication) {
        print("⏸ Приложение ушло в фон")
        
        // Сохранить состояние, остановить таймеры, приостановить воспроизведение
    }
    
    // 4. Получение push-уведомления (foreground + background fetch)
    func application(_ application: UIApplication,
                     didReceiveRemoteNotification userInfo: [AnyHashable : Any],
                     fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {
        
        print("📬 Получено push: \(userInfo)")
        
        // Обработка данных
        if let aps = userInfo["aps"] as? [String: Any],
           let alert = aps["alert"] as? [String: Any],
           let title = alert["title"] as? String {
            print("Заголовок пуша: \(title)")
        }
        
        completionHandler(.newData)
    }
    
    // 5. Успешная регистрация на push
    func application(_ application: UIApplication,
                     didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        let token = deviceToken.map { String(format: "%02.2hhx", $0) }.joined()
        print("Device Token: \(token)")
        
        // Отправить токен на сервер
    }
    
    // 6. Ошибка регистрации на push
    func application(_ application: UIApplication,
                     didFailToRegisterForRemoteNotificationsWithError error: Error) {
        print("❌ Ошибка регистрации push: \(error)")
    }
}
```

### Короткий чек-лист «Что нужно сделать с AppDelegate в 2026»

1. Поставить `@main` над классом  
2. Реализовать `didFinishLaunchingWithOptions` — здесь вся инициализация  
3. Добавить обработку состояний: `DidBecomeActive`, `DidEnterBackground`, `WillTerminate`  
4. Настроить push (регистрация + получение)  
5. (Опционально) добавить поддержку URL-схем, Universal Links, Handoff, Siri Shortcuts  
6. Использовать **SceneDelegate** / **SwiftUI App** для управления окнами и сценами (если проект не чистый legacy UIKit)

### Короткий девиз 2026

> AppDelegate — это «мозг» всего приложения: он ловит запуск, фон, уведомления, токены и все глобальные события.  
> В 2026 году в новых проектах его роль уменьшилась (многое ушло в SceneDelegate и @main), но он всё ещё **обязателен** для push, Firebase, аналитики и глобальной инициализации.  
> Пиши минимум кода, но обязательно обрабатывай ключевые состояния.

Удачи с правильным запуском и жизненным циклом приложения в Swift! 🚀