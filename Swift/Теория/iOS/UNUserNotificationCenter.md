**`UNUserNotificationCenter`** — это центральный класс в фреймворке **[[UserNotifications]]**, который отвечает за всю работу с **локальными и push-уведомлениями** в [[iOS]], iPadOS, macOS, watchOS и tvOS (начиная с iOS 10).

Это **единственный официальный способ**:
- запрашивать разрешение на уведомления,
- планировать локальные уведомления,
- управлять уже запланированными/доставленными уведомлениями,
- обрабатывать действия пользователя с уведомлениями.

### Основные возможности UNUserNotificationCenter (актуально на 2026 год)

| Задача                                      | Основной метод / свойство                                   | Ключевые замечания 2026 |
|---------------------------------------------|-------------------------------------------------------------|--------------------------|
| Запрос разрешения на уведомления            | `requestAuthorization(options:completionHandler:)`          | Обязательно! iOS 10+     |
| Проверка текущего статуса разрешения        | `getNotificationSettings(completionHandler:)`               | `.authorizationStatus`   |
| Планирование локального уведомления         | `add(_:withCompletionHandler:)`                             | `UNNotificationRequest`  |
| Удаление запланированных уведомлений        | `removePendingNotificationRequests(withIdentifiers:)`       | По ID или все            |
| Удаление доставленных уведомлений           | `removeDeliveredNotifications(withIdentifiers:)`            | Из центра уведомлений    |
| Получение списка запланированных            | `getPendingNotificationRequests(completionHandler:)`        | Редко, но полезно        |
| Получение доставленных уведомлений          | `getDeliveredNotifications(completionHandler:)`             | iOS 10+                  |
| Обработка действий с уведомлением          | Через `UNUserNotificationCenterDelegate`                    | `didReceive response`    |

### Минимальный рабочий пример (самый частый паттерн 2026)

```swift
import UserNotifications

@MainActor
final class NotificationManager: NSObject, UNUserNotificationCenterDelegate {
    
    static let shared = NotificationManager()
    
    private override init() {
        super.init()
        UNUserNotificationCenter.current().delegate = self
    }
    
    func requestAuthorization() async throws -> Bool {
        let options: UNAuthorizationOptions = [.alert, .sound, .badge]
        return try await UNUserNotificationCenter.current().requestAuthorization(options: options)
    }
    
    func scheduleLocalNotification(title: String, body: String, timeInterval: TimeInterval) {
        let content = UNMutableNotificationContent()
        content.title = title
        content.body = body
        content.sound = .default
        
        // Триггер через время
        let trigger = UNTimeIntervalNotificationTrigger(timeInterval: timeInterval, repeats: false)
        
        let request = UNNotificationRequest(identifier: UUID().uuidString,
                                           content: content,
                                           trigger: trigger)
        
        UNUserNotificationCenter.current().add(request) { error in
            if let error {
                print("Ошибка планирования:", error.localizedDescription)
            }
        }
    }
    
    // Делегат: уведомление пришло, когда приложение на переднем плане
    func userNotificationCenter(_ center: UNUserNotificationCenter,
                                willPresent notification: UNNotification,
                                withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        
        // Показываем уведомление даже если приложение открыто
        completionHandler([.banner, .sound])
    }
    
    // Делегат: пользователь взаимодействовал с уведомлением
    func userNotificationCenter(_ center: UNUserNotificationCenter,
                                didReceive response: UNNotificationResponse,
                                withCompletionHandler completionHandler: @escaping () -> Void) {
        
        let userInfo = response.notification.request.content.userInfo
        print("Пользователь взаимодействовал с уведомлением:", userInfo)
        
        // Обработка действия (если есть custom actions)
        if response.actionIdentifier == "REPLY_ACTION" {
            // ...
        }
        
        completionHandler()
    }
}
```

### Как использовать в реальном приложении (рекомендуемый подход)

```swift
class AppDelegate: UIResponder, UIApplicationDelegate {
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        UNUserNotificationCenter.current().delegate = NotificationManager.shared
        
        Task {
            do {
                let granted = try await NotificationManager.shared.requestAuthorization()
                if granted {
                    print("Разрешение получено")
                }
            } catch {
                print("Ошибка запроса разрешения:", error)
            }
        }
        
        return true
    }
}
```

### Ключевые Info.plist ключи (обязательно!)

```xml
<!-- Для уведомлений -->
<key>NSUserNotificationsUsageDescription</key>
<string>Приложение хочет отправлять вам уведомления о новых сообщениях и напоминаниях.</string>

<!-- Если используете critical alerts (редко) -->
<key>UIBackgroundModes</key>
<array>
    <string>remote-notification</string>
</array>
```

### Лучшие практики UNUserNotificationCenter в 2026

- **Всегда** запрашивайте разрешение **один раз** при первом запуске (или при первой необходимости)  
- **Используйте** [[async]]/[[await]] версию `requestAuthorization` (iOS 15+) — чище и современнее  
- **Обязательно** реализуйте делегат и оба метода (`willPresent` и `didReceive`)  
- **Держите делегат** как отдельный объект (чаще всего singleton или сервис-класс)  
- **Для [[SwiftUI]]** — создавайте `@ObservableObject` обёртку и используйте `.task` / `.onReceive`  
- **Для фоновой обработки** — используйте [[UNNotificationServiceExtension]] и [[UNNotificationContentExtension]]  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для уведомлений  
- **Документируйте** — пишите комментарий «UNUserNotificationCenterDelegate — обработка локальных и push-уведомлений»

**Короткий итог 2026**:
> `UNUserNotificationCenter` — центральный менеджер всех уведомлений в iOS.  
> В 2026 году:  
> - запрашивайте разрешение **один раз** через `requestAuthorization`  
> - реализуйте делегат и обрабатывайте **оба** метода  
> - используйте `async/await` для запроса разрешения  
> - всегда проверяйте `canSendMail()` аналогично — `authorizationStatus`  
> Это **единственный** способ полноценно работать с уведомлениями на iOS.
