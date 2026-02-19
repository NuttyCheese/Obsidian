**UserNotifications** — это фреймворк в [[iOS]], iPadOS, macOS, watchOS и tvOS, который отвечает за работу с **локальными** и **push-уведомлениями** (remote notifications) начиная с iOS 10.

Центральный класс фреймворка — **`UNUserNotificationCenter`**.

### Основные возможности UserNotifications (актуально на 2026 год)

| Задача                                      | Основной класс / метод                                      | Ключевые замечания 2026 |
|---------------------------------------------|-------------------------------------------------------------|--------------------------|
| Запрос разрешения на уведомления            | `UNUserNotificationCenter.current().requestAuthorization`   | Обязательно! Строгое поведение в iOS 18+ |
| Проверка текущего статуса разрешения        | `getNotificationSettings`                                   | `.authorizationStatus`: `.authorized`, `.denied`, `.notDetermined`, `.provisional`, `.ephemeral` |
| Планирование локального уведомления         | `add(_:withCompletionHandler:)`                             | `UNMutableNotificationContent` + триггер |
| Управление запланированными уведомлениями   | `getPendingNotificationRequests`, `removePending…`          | До 64 локальных уведомлений на приложение |
| Управление доставленными уведомлениями      | `getDeliveredNotifications`, `removeDelivered…`             | Удаление из центра уведомлений |
| Обработка действий пользователя             | `UNUserNotificationCenterDelegate`                          | `didReceive response`, `willPresent` |
| Critical alerts (критические уведомления)   | `criticalAlert` в `UNAuthorizationOptions`                  | Требует entitlement + отдельное разрешение |
| Notification Service Extension              | Отдельное расширение для модификации контента перед доставкой | Rich push, шифрование, обработка медиа |
| Notification Content Extension              | Кастомный UI для уведомлений (rich notifications)           | Интерактивные уведомления |

### Минимальный рабочий пример (современный стиль 2026)

```swift
import UserNotifications

@MainActor
final class NotificationService: NSObject, UNUserNotificationCenterDelegate {
    
    static let shared = NotificationService()
    
    private override init() {
        super.init()
        UNUserNotificationCenter.current().delegate = self
    }
    
    func requestAuthorization() async throws -> Bool {
        let options: UNAuthorizationOptions = [.alert, .sound, .badge]
        return try await UNUserNotificationCenter.current().requestAuthorization(options: options)
    }
    
    func scheduleReminder(title: String, body: String, timeInterval: TimeInterval) {
        let content = UNMutableNotificationContent()
        content.title = title
        content.body = body
        content.sound = .default
        // content.badge = 1 // если нужно
        
        let trigger = UNTimeIntervalNotificationTrigger(timeInterval: timeInterval, repeats: false)
        
        let request = UNNotificationRequest(identifier: UUID().uuidString,
                                           content: content,
                                           trigger: trigger)
        
        UNUserNotificationCenter.current().add(request) { error in
            if let error {
                print("Ошибка планирования уведомления:", error.localizedDescription)
            }
        }
    }
    
    // Уведомление пришло, когда приложение на переднем плане
    func userNotificationCenter(_ center: UNUserNotificationCenter,
                                willPresent notification: UNNotification,
                                withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        
        // Показываем баннер даже если приложение открыто
        completionHandler([.banner, .sound, .badge])
    }
    
    // Пользователь взаимодействовал с уведомлением
    func userNotificationCenter(_ center: UNUserNotificationCenter,
                                didReceive response: UNNotificationResponse,
                                withCompletionHandler completionHandler: @escaping () -> Void) {
        
        let userInfo = response.notification.request.content.userInfo
        print("Действие с уведомлением:", userInfo)
        
        // Обработка custom action
        if response.actionIdentifier == UNNotificationDefaultActionIdentifier {
            print("Открыто уведомление")
        } else if response.actionIdentifier == "REPLY_ACTION" {
            // Обработка текста ответа
            if let response = response as? UNTextInputNotificationResponse {
                print("Ответ пользователя:", response.userText)
            }
        }
        
        completionHandler()
    }
}
```

### Как правильно интегрировать в приложение (рекомендуемый подход 2026)

```swift
class AppDelegate: UIResponder, UIApplicationDelegate {
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        UNUserNotificationCenter.current().delegate = NotificationService.shared
        
        Task {
            do {
                let granted = try await NotificationService.shared.requestAuthorization()
                if granted {
                    print("Разрешение на уведомления получено")
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
<!-- Основное описание для уведомлений -->
<key>NSUserNotificationsUsageDescription</key>
<string>Приложение хочет отправлять вам уведомления о новых сообщениях, напоминаниях и акциях.</string>

<!-- Если используете critical alerts -->
<key>UIBackgroundModes</key>
<array>
    <string>remote-notification</string>
</array>

<!-- Для критических уведомлений (редко) -->
<key>NSUserNotificationUsageDescription</key>
<string>Критические уведомления о безопасности и важных событиях.</string>
```

### Лучшие практики UNUserNotificationCenter в 2026

- **Запрашивайте разрешение** **один раз** (лучше при первом запуске или при первой необходимости)  
- **Используйте** `async/await` версию `requestAuthorization` (iOS 15+) — современный и чистый стиль  
- **Обязательно** реализуйте делегат и оба метода (`willPresent` и `didReceive`)  
- **Держите делегат** как отдельный объект (singleton или сервис-класс)  
- **Для SwiftUI** — создавайте `@ObservableObject` обёртку и используйте `.task` / `.onReceive`  
- **Для фоновой обработки** — используйте `UNNotificationServiceExtension` и `UNNotificationContentExtension`  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для всех функций UserNotifications  
- **Документируйте** — пишите комментарий «UNUserNotificationCenterDelegate — обработка локальных и push-уведомлений»

**Короткий итог 2026**:
> `UNUserNotificationCenter` — центральный менеджер всех уведомлений в iOS.  
> В 2026 году:  
> - запрашивайте разрешение **один раз** через `requestAuthorization`  
> - реализуйте делегат и обрабатывайте **оба** метода  
> - используйте `async/await` для запроса разрешения  
> - всегда проверяйте `canSendMail()` аналогично — `authorizationStatus`  
> Это **единственный** способ полноценно работать с уведомлениями на iOS.

Удачи с надёжными, timely и user-friendly уведомлениями в твоём приложении! 🔔