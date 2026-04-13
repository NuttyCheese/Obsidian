#unnotificationcontent #usernotifications #push-notifications #local-notifications #rich-notifications #ios #swift #notification-content #attachments #category #threading #ios-10

---
(содержимое уведомления / контент уведомления)

**UNNotificationContent** — это **неизменяемый** (immutable) объект в фреймворке UserNotifications (с iOS 10+), который содержит **всё, что система должна показать пользователю** в уведомлении:

- заголовок, подзаголовок, основной текст
- звук, значок (badge), категория
- медиа-вложения (фото, видео, аудио)
- группировку (thread), summary, кастомные данные

Он используется в двух вариантах:

- **UNNotificationContent** — неизменяемый, приходит в готовом виде (из APNs или локального запроса)
- **UNMutableNotificationContent** — изменяемый, создаётся вами для локальных уведомлений или модификации в **[[UNNotificationServiceExtension]]**

### Основные свойства UNNotificationContent

| Свойство                | Тип                        | Что задаёт / зачем нужно                       | Изменяется в Mutable? | Примечание 2026                             |
| ----------------------- | -------------------------- | ---------------------------------------------- | --------------------- | ------------------------------------------- |
| title                   | [[String]]                 | Главный заголовок уведомления                  | Да                    | Поддерживает эмодзи, локализацию            |
| subtitle                | String                     | Подзаголовок (часто имя отправителя)           | Да                    | Отображается мелким шрифтом                 |
| body                    | String                     | Основной текст уведомления                     | Да                    | До ~400 символов                            |
| sound                   | UNNotificationSound?       | Звук уведомления (`default` или кастомный)     | Да                    | `UNNotificationSound(named: "mysound.caf")` |
| badge                   | [[NSNumber]]?              | Число на иконке приложения                     | Да                    | `nil` = не менять                           |
| attachments             | [UNNotificationAttachment] | Медиа-вложения (изображения, видео, аудио)     | Да                    | Добавляются через Service Extension         |
| categoryIdentifier      | String                     | Категория (определяет кнопки действий)         | Да                    | Должна быть зарегистрирована                |
| threadIdentifier        | String                     | Группировка уведомлений (чат/тред)             | Да                    | Один тред = одна группа                     |
| summaryArgument         | String                     | Аргумент для Summary (iOS 15+)                 | Да                    | "5 новых сообщений"                         |
| summaryArgumentCount    | [[Int]]                    | Количество элементов в summary                 | Да                    | Для множественных уведомлений               |
| targetContentIdentifier | String?                    | Связь с Live Activity (iOS 16+)                | Да                    | Для динамических активностей                |
| userInfo                | [AnyHashable: Any]         | Произвольные данные для приложения             | Да                    | Deep linking, аналитика                     |
| launchImageName         | String?                    | Имя изображения для launch screen при открытии | Да                    | Редко используется                          |

### Создание и модификация содержимого

#### 1. Локальное уведомление (самый частый случай)

```swift
let content = UNMutableNotificationContent()
content.title = "Новое сообщение"
content.subtitle = "Алексей"
content.body = "Привет! Как дела? 😊"
content.sound = .default
content.badge = 3
content.threadIdentifier = "chat-alexey-123"
content.categoryIdentifier = "chat-message"
content.userInfo = ["chat_id": "alexey-123", "message_id": "456"]

// Добавление вложения (если есть файл в бандле)
if let url = Bundle.main.url(forResource: "avatar", withExtension: "jpg") {
    let attachment = try? UNNotificationAttachment(identifier: "avatar",
                                                   url: url,
                                                   options: nil)
    if let attachment {
        content.attachments = [attachment]
    }
}

// Планирование
let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 5, repeats: false)
let request = UNNotificationRequest(identifier: UUID().uuidString,
                                    content: content,
                                    trigger: trigger)

UNUserNotificationCenter.current().add(request)
```

#### 2. Модификация в Notification Service Extension

```swift
class NotificationService: UNNotificationServiceExtension {
    
    var contentHandler: ((UNNotificationContent) -> Void)?
    var bestAttemptContent: UNMutableNotificationContent?
    
    override func didReceive(_ request: UNNotificationRequest,
                            withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void) {
        
        self.contentHandler = contentHandler
        bestAttemptContent = (request.content.mutableCopy() as? UNMutableNotificationContent)
        
        guard let bestAttemptContent else {
            contentHandler(request.content)
            return
        }
        
        // Изменяем содержимое
        bestAttemptContent.title = "\(bestAttemptContent.title) 📸"
        bestAttemptContent.body = "\(bestAttemptContent.body)\n(отредактировано расширением)"
        
        // Добавляем вложение по URL из push
        if let imageURLString = bestAttemptContent.userInfo["image_url"] as? String,
           let url = URL(string: imageURLString) {
            
            URLSession.shared.downloadTask(with: url) { location, _, error in
                guard let location, error == nil else {
                    contentHandler(bestAttemptContent)
                    return
                }
                
                let attachment = try? UNNotificationAttachment(identifier: "image",
                                                               url: location,
                                                               options: nil)
                if let attachment {
                    bestAttemptContent.attachments = [attachment]
                }
                
                contentHandler(bestAttemptContent)
            }.resume()
        } else {
            contentHandler(bestAttemptContent)
        }
    }
    
    override func serviceExtensionTimeWillExpire() {
        if let contentHandler, let bestAttemptContent {
            contentHandler(bestAttemptContent)
        }
    }
}
```

### Лучшие практики UNNotificationContent в 2026 году

- **Всегда** используйте `UNMutableNotificationContent` для создания и модификации  
- **threadIdentifier** — обязательно для группировки чатов/тем  
- **attachments** — добавляйте через Service Extension (скачивание по URL)  
- **mutable-content: 1** — обязательно в payload для Service Extension  
- **categoryIdentifier** — регистрируйте заранее для кнопок действий  
- **userInfo** — используйте для deep linking и передачи данных в приложение  
- **Тестирование** — локальные уведомления + симуляция push через консоль  
- **Для iOS 18+** — поддержка Live Activities, но `UNNotificationContent` остаётся основой  
- **Для [[SwiftUI]]** — всё через UserNotifications, но UI можно кастомизировать в Content Extension  

**Короткий итог 2026**:
> **UNNotificationContent** — это **всё содержимое** уведомления (текст, звук, вложения, категория, данные).  
> Junior: "То, что увидит пользователь в уведомлении".  
> Middle: используйте `UNMutableNotificationContent` для создания; добавляйте вложения через Service Extension.  
> Senior: threadIdentifier для группировки, mutable-content для модификации, userInfo для deep linking, тестируйте локально, комбинируйте с категориями и Content Extension.  
