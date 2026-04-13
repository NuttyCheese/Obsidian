#unnotificationrequest #usernotifications #push-notifications #local-notifications #remote-notifications #ios #swift #notification-center #mutable-content #attachments #category #user-notifications-ui

---
(запрос на уведомление / уведомительный запрос)

**UNNotificationRequest** — это основной объект в фреймворке UserNotifications (с iOS 10+), который описывает **одно конкретное уведомление**, которое нужно показать пользователю — либо локальное (создано приложением), либо удалённое (пришло через APNs).

Он состоит из двух обязательных частей:

1. **identifier** — уникальный строковый ID уведомления  
2. **content** — содержимое ([[UNNotificationContent]]): заголовок, текст, звук, вложения, категория и т.д.  
3. **trigger** — условие показа ([[UNNotificationTrigger]]): по времени, по геолокации, по push от сервера и т.д.

Без триггера уведомление не покажется никогда (кроме немедленного показа через [[UNUserNotificationCenter]]).

### Основные способы создания UNNotificationRequest

| Тип уведомления         | Как создаётся                                      | Триггер                              | Самый частый сценарий 2026 |
|-------------------------|-----------------------------------------------------|--------------------------------------|-----------------------------|
| Локальное (по времени)  | `UNNotificationRequest(identifier:content:trigger:)` | `UNTimeIntervalNotificationTrigger`  | Напоминания, будильники, ежедневные задачи |
| Локальное (календарь)   | То же                                               | `UNCalendarNotificationTrigger`      | Ежедневные/еженедельные события |
| Локальное (геолокация)  | То же                                               | `UNLocationNotificationTrigger`      | Прийти домой → "Добро пожаловать" |
| Удалённое (push)        | Сервер отправляет payload → система создаёт request | nil (trigger = nil)                  | Сообщения, лайки, заказы |
| Немедленное (foreground) | `UNUserNotificationCenter.current().add(request)` с trigger = nil | nil                                  | Показать уведомление сразу (редко) |

### Структура и ключевые свойства (2026)

```swift
let request = UNNotificationRequest(
    identifier: "reminder-123",                    // уникальный ID (String)
    content: content,                              // UNNotificationContent / UNMutableNotificationContent
    trigger: trigger                               // UNNotificationTrigger? (nil = сразу)
)
```

**UNNotificationContent** (самая важная часть):

| Свойство                | Тип                        | Что задаёт                             | Примечание 2026                             |
| ----------------------- | -------------------------- | -------------------------------------- | ------------------------------------------- |
| title                   | [[String]]                 | Заголовок                              | Поддерживает эмодзи, локализацию            |
| subtitle                | String                     | Подзаголовок                           | Часто имя отправителя                       |
| body                    | String                     | Основной текст                         | До ~400 символов                            |
| sound                   | UNNotificationSound?       | Звук (default / custom)                | `UNNotificationSound(named: "mysound.caf")` |
| badge                   | [[NSNumber]]?              | Число на иконке приложения             | nil = не менять                             |
| attachments             | [UNNotificationAttachment] | Медиа-вложения (фото, видео, аудио)    | Добавляются через Service Extension         |
| categoryIdentifier      | String                     | Категория (определяет кнопки действий) | Должна быть зарегистрирована                |
| userInfo                | [AnyHashable: Any]         | Произвольные данные для приложения     | Deep linking, аналитика                     |
| threadIdentifier        | String                     | Группировка уведомлений (чат/тред)     | Один тред = одна группа                     |
| summaryArgument         | String                     | Для Summary (iOS 15+)                  | "5 новых сообщений"                         |
| targetContentIdentifier | String?                    | Для Live Activities ([[iOS]] 16+)      | Связь с активностью                         |

### Самый популярный паттерн 2026 года  
(Локальное уведомление + богатое содержимое + действия)

```swift
import UserNotifications

func scheduleRichReminder() {
    let center = UNUserNotificationCenter.current()
    
    // 1. Проверяем и запрашиваем разрешение (обычно в AppDelegate / SceneDelegate)
    center.requestAuthorization(options: [.alert, .sound, .badge]) { granted, error in
        guard granted else { return }
    }
    
    // 2. Создаём содержимое
    let content = UNMutableNotificationContent()
    content.title = "Время пить воду 💧"
    content.body = "Ты уже 2 часа не пил! Давай сделаем глоток?"
    content.sound = .default
    content.badge = 1
    content.threadIdentifier = "hydration-reminder"
    content.categoryIdentifier = "hydration-category"  // для кнопок действий
    
    // Добавляем картинку (должна быть в бандле расширения или скачана заранее)
    if let url = Bundle.main.url(forResource: "water-glass", withExtension: "jpg") {
        let attachment = try? UNNotificationAttachment(identifier: "water",
                                                       url: url,
                                                       options: nil)
        if let attachment {
            content.attachments = [attachment]
        }
    }
    
    // 3. Триггер — через 2 часа
    let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 7200, repeats: false)
    
    // 4. Создаём запрос
    let request = UNNotificationRequest(identifier: UUID().uuidString,
                                        content: content,
                                        trigger: trigger)
    
    // 5. Планируем
    center.add(request) { error in
        if let error {
            print("Ошибка планирования: \(error)")
        }
    }
}
```

### Регистрация категории с действиями (обязательно для кнопок)

```swift
func registerNotificationCategories() {
    let replyAction = UNTextInputNotificationAction(identifier: "reply-action",
                                                    title: "Ответить",
                                                    options: [.authenticationRequired])
    
    let likeAction = UNNotificationAction(identifier: "like-action",
                                          title: "Лайк ❤️",
                                          options: [])
    
    let category = UNNotificationCategory(identifier: "hydration-category",
                                          actions: [replyAction, likeAction],
                                          intentIdentifiers: [],
                                          options: [.customDismissAction])
    
    UNUserNotificationCenter.current().setNotificationCategories([category])
}
```

### Лучшие практики UNNotificationRequest в 2026 году

- **identifier** — всегда уникальный ([[UUID]]().uuidString) для локальных, для remote — можно использовать server-side ID  
- **mutable-content: 1** — обязательно для Service Extension (модификация контента)  
- **categoryIdentifier** — регистрируйте заранее через `setNotificationCategories`  
- **attachments** — добавляйте через Service Extension (скачивание по URL)  
- **threadIdentifier** — группируйте уведомления по чатам/темам  
- **summaryArgument** — используйте для iOS 15+ Summary (количество + аргумент)  
- **Тестирование** — используйте локальные запросы + консоль → симуляция push  
- **Для SwiftUI** — уведомления всё ещё через UserNotifications, но UI можно сделать через SwiftUI Hosting в Content Extension  
- **Для iOS 18+** — поддержка Live Activities, но `UNNotificationRequest` остаётся основой для классических уведомлений

**Короткий итог 2026**:
> **UNNotificationRequest** — это **одно уведомление** (локальное или удалённое), которое нужно показать.  
> Junior: "Сообщение, которое появится на экране блокировки".  
> Middle: состоит из identifier + content + trigger; требует разрешения; поддерживает вложения и категории.  
> Senior: используйте `mutable-content: 1` + Service Extension для медиа; регистрируйте категории для действий; группируйте через threadIdentifier; тестируйте локально.  
