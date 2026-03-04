#unnotificationresponse #usernotifications #push-notifications #notification-actions #user-notifications #ios #swift #interactive-notifications #custom-actions #background-handling #ios-10 

---
(ответ на уведомление / уведомительный ответ)

**UNNotificationResponse** — это объект, который система передаёт вашему приложению, когда пользователь **взаимодействует** с уведомлением:

- нажимает на само уведомление (открывает приложение)
- нажимает на одну из **кастомных кнопок действий** (Action)
- отвечает текстом на **текстовое поле ввода** в уведомлении (Text Input Action)

Он появился в **iOS 10** (2016) и остаётся центральным объектом для обработки **интерактивных уведомлений** в 2026 году.

### Когда и где вы получаете UNNotificationResponse

| Сценарий                                      | Где обрабатывается                                       | Тип объекта | Самый частый случай 2026 |
|-----------------------------------------------|-----------------------------------------------------------|-------------|---------------------------|
| Пользователь нажал на уведомление             | `application(_:didReceiveRemoteNotification:fetchCompletionHandler:)` или `userNotificationCenter(_:didReceive:withCompletionHandler:)` | `UNNotificationResponse` | Открытие приложения из уведомления |
| Пользователь нажал на кастомную кнопку        | `userNotificationCenter(_:didReceive:withCompletionHandler:)` | `UNNotificationResponse` | "Ответить", "Лайк", "Отложить" |
| Пользователь ввёл текст в уведомлении         | То же                                                     | `UNTextInputNotificationResponse` (подкласс) | Быстрый ответ в чате |
| Уведомление открыто в фоне / quit-состоянии   | `application(_:didReceiveRemoteNotification:fetchCompletionHandler:)` (legacy) или новый метод | `UNNotificationResponse` | Обработка silent push + actions |

### Структура UNNotificationResponse

```swift
class UNNotificationResponse: NSObject, NSSecureCoding {
    var actionIdentifier: String              // "com.apple.UNNotificationDefaultActionIdentifier" или ваш ID
    var notification: UNNotification          // само уведомление (содержимое + запрос)
}

class UNTextInputNotificationResponse: UNNotificationResponse {
    var userText: String                      // текст, введённый пользователем
}
```

**Ключевые значения actionIdentifier:**

| Значение                                          | Что значит                                           | Когда приходит |
|---------------------------------------------------|------------------------------------------------------|----------------|
| `UNNotificationDefaultActionIdentifier`           | Пользователь просто нажал на уведомление             | Открытие приложения |
| `UNNotificationDismissActionIdentifier`           | Пользователь смахнул уведомление (удалил)            | Редко обрабатывается |
| Ваш кастомный ID ("reply-action", "like-action")  | Пользователь нажал на вашу кнопку действия           | Самый интересный случай |

### Самый популярный и рекомендуемый паттерн 2026 года  
(Обработка действий из уведомления + открытие конкретного экрана)

```swift
import UserNotifications

class AppDelegate: UIResponder, UIApplicationDelegate, UNUserNotificationCenterDelegate {
    
    func userNotificationCenter(_ center: UNUserNotificationCenter,
                                didReceive response: UNNotificationResponse,
                                withCompletionHandler completionHandler: @escaping () -> Void) {
        
        let userInfo = response.notification.request.content.userInfo
        let actionID = response.actionIdentifier
        
        // 1. Общий случай — просто открыли уведомление
        if actionID == UNNotificationDefaultActionIdentifier {
            handleNotificationOpen(userInfo: userInfo)
        }
        
        // 2. Кастомные действия
        switch actionID {
        case "reply-action":
            if let textResponse = response as? UNTextInputNotificationResponse {
                let userText = textResponse.userText
                handleReply(userText: userText, userInfo: userInfo)
            }
            
        case "like-action":
            handleLike(userInfo: userInfo)
            
        case "snooze-action":
            handleSnooze(userInfo: userInfo)
            
        default:
            break
        }
        
        // 3. Обязательно вызываем completionHandler
        completionHandler()
    }
    
    private func handleNotificationOpen(userInfo: [AnyHashable: Any]) {
        // Пример: deep linking — открыть чат с конкретным пользователем
        if let chatID = userInfo["chat_id"] as? String {
            // Перейти на экран чата
            if let windowScene = UIApplication.shared.connectedScenes.first as? UIWindowScene,
               let rootVC = windowScene.windows.first?.rootViewController as? UITabBarController {
                rootVC.selectedIndex = 0 // вкладка сообщений
                // Дальше — навигация к чату с chatID
            }
        }
    }
    
    private func handleReply(userText: String, userInfo: [AnyHashable: Any]) {
        // Отправить ответ на сервер
        if let chatID = userInfo["chat_id"] as? String {
            // API call: отправить userText в чат chatID
            print("Ответ в чат \(chatID): \(userText)")
        }
    }
    
    private func handleLike(userInfo: [AnyHashable: Any]) {
        // Отправить лайк на сервер
    }
    
    private func handleSnooze(userInfo: [AnyHashable: Any]) {
        // Запланировать повторное уведомление через 10 минут
        let content = UNMutableNotificationContent()
        content.title = "Напоминание"
        content.body = "Вы отложили задачу"
        
        let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 600, repeats: false)
        let request = UNNotificationRequest(identifier: UUID().uuidString,
                                            content: content,
                                            trigger: trigger)
        
        UNUserNotificationCenter.current().add(request)
    }
}
```

### Регистрация категорий и действий (обязательно)

```swift
func registerNotificationCategories() {
    let replyAction = UNTextInputNotificationAction(
        identifier: "reply-action",
        title: "Быстрый ответ",
        options: [.authenticationRequired]
    )
    
    let likeAction = UNNotificationAction(
        identifier: "like-action",
        title: "Лайк ❤️",
        options: []
    )
    
    let snoozeAction = UNNotificationAction(
        identifier: "snooze-action",
        title: "Отложить на 10 мин",
        options: []
    )
    
    let chatCategory = UNNotificationCategory(
        identifier: "chat-message-category",
        actions: [replyAction, likeAction, snoozeAction],
        intentIdentifiers: [],
        options: [.customDismissAction, .hiddenPreviewsShowTitle]
    )
    
    UNUserNotificationCenter.current().setNotificationCategories([chatCategory])
}
```

### Лучшие практики UNNotificationResponse в 2026 году

- **Всегда вызывайте completionHandler** в конце `didReceive` — иначе приложение зависнет  
- **Используйте `threadIdentifier`** в content для группировки уведомлений  
- **Для text input** — обрабатывайте `UNTextInputNotificationResponse.userText`  
- **[[Deep link]]ing** — передавайте параметры через `userInfo` и обрабатывайте в `didReceive`  
- **Безопасность** — проверяйте `userInfo` на наличие ожидаемых ключей  
- **Для iOS 18+** — поддержка Live Activities, но `UNNotificationResponse` остаётся основой для действий  
- **Тестирование** — используйте локальные уведомления + симуляцию push через консоль  
- **Для [[SwiftUI]]** — обрабатывайте в `@UIApplicationDelegateAdaptor` или `UNUserNotificationCenterDelegate`  

**Короткий итог 2026**:
> **UNNotificationResponse** — объект, который приходит в приложение, когда пользователь **взаимодействует** с уведомлением (нажал на него или на кнопку действия).  
> Junior: "Сообщает, что сделал пользователь с уведомлением".  
> Middle: содержит `actionIdentifier`, `notification` и (при вводе текста) `userText`.  
> Senior: используйте для deep linking, обработки действий, отправки ответов на сервер; вызывайте `completionHandler`; комбинируйте с категориями и Service/Content Extensions.  
