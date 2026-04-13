**`UNUserNotificationCenterDelegate`** — это протокол в фреймворке **[[UserNotifications]]**, который определяет методы-делегаты для обработки событий, связанных с **локальными и удалёнными (push) уведомлениями**.

Он обязателен, если вы хотите:
- показывать уведомления, когда приложение находится **на переднем плане**,
- обрабатывать действия пользователя с уведомлением (тап, ответ на текст, нажатие кнопки),
- получать уведомления о доставке/ошибках.

### Основные методы протокола (актуальные на 2026 год)

| Метод делегата                                                 | Когда вызывается                                                                         | Что обязательно делать внутри метода                                                    | Самый частый сценарий                    |
| -------------------------------------------------------------- | ---------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | ---------------------------------------- |
| `userNotificationCenter(_:willPresent:withCompletionHandler:)` | Уведомление пришло, пока приложение **на переднем плане**                                | Вызвать completionHandler с нужными опциями презентации (.banner, .sound, .list и т.д.) | Показ баннера при активном приложении    |
| `userNotificationCenter(_:didReceive:withCompletionHandler:)`  | Пользователь **взаимодействовал** с уведомлением (тапнул, ответил текстом, нажал кнопку) | Вызвать completionHandler() после обработки действия                                    | Открытие экрана, обработка [[deep link]] |
| `userNotificationCenter(_:openSettingsFor:)`                   | Пользователь нажал на кнопку «Настройки» в уведомлении ([[iOS]] 12+)                     | Открыть нужный экран настроек приложения (опционально)                                  | Редко используется                       |

### Полный рекомендуемый шаблон реализации (2026 стандарт)

```swift
import UserNotifications

@MainActor
final class NotificationDelegate: NSObject, UNUserNotificationCenterDelegate {
    
    static let shared = NotificationDelegate()
    
    private override init() {
        super.init()
        UNUserNotificationCenter.current().delegate = self
    }
    
    // Уведомление пришло, приложение активно
    func userNotificationCenter(_ center: UNUserNotificationCenter,
                                willPresent notification: UNNotification,
                                withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        
        // Показываем баннер и звук даже если приложение открыто
        // Можно добавить логику: не показывать, если пользователь в чате и т.д.
        completionHandler([.banner, .sound, .badge])
        
        // Пример: логируем
        print("Уведомление на переднем плане:", notification.request.content.title)
    }
    
    // Пользователь взаимодействовал с уведомлением
    func userNotificationCenter(_ center: UNUserNotificationCenter,
                                didReceive response: UNNotificationResponse,
                                withCompletionHandler completionHandler: @escaping () -> Void) {
        
        let content = response.notification.request.content
        let userInfo = content.userInfo
        
        print("Действие с уведомлением:", content.title, userInfo)
        
        // Обработка стандартного тапа
        if response.actionIdentifier == UNNotificationDefaultActionIdentifier {
            // Открываем нужный экран
            handleNotificationTap(userInfo: userInfo)
        }
        
        // Обработка кастомной кнопки
        else if response.actionIdentifier == "REPLY_ACTION" {
            if let textResponse = response as? UNTextInputNotificationResponse {
                print("Пользователь ответил:", textResponse.userText)
                // Отправляем ответ на сервер
            }
        }
        
        // Обработка кнопки "Отложить"
        else if response.actionIdentifier == "SNOOZE_ACTION" {
            print("Уведомление отложено")
        }
        
        // Важно: вызываем completionHandler после обработки
        completionHandler()
    }
    
    private func handleNotificationTap(userInfo: [AnyHashable: Any]) {
        // Пример: deep link или открытие конкретного экрана
        if let orderId = userInfo["orderId"] as? String {
            // Открываем экран заказа
            print("Открываем заказ:", orderId)
        }
    }
}
```

### Как правильно подключить делегат (рекомендуемый способ)

```swift
// В AppDelegate или SceneDelegate
func application(_ application: UIApplication,
                 didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    
    UNUserNotificationCenter.current().delegate = NotificationDelegate.shared
    
    // Запрос разрешения (лучше через async/await)
    Task {
        do {
            let granted = try await UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .sound, .badge])
            if granted {
                print("Разрешение получено")
            }
        } catch {
            print("Ошибка запроса:", error)
        }
    }
    
    return true
}
```

### Лучшие практики UNUserNotificationCenterDelegate в 2026 году

- **Всегда** устанавливайте делегат **один раз** при запуске приложения (в `didFinishLaunchingWithOptions`)  
- **Обязательно** вызывайте `completionHandler` в обоих методах — иначе уведомления могут "зависнуть"  
- **В `willPresent`** — возвращайте `.banner` + `.sound`, если хотите показывать уведомление при активном приложении  
- **В `didReceive`** — обрабатывайте `actionIdentifier` (включая `UNNotificationDefaultActionIdentifier`) и текст ответа  
- **Для SwiftUI** — создавайте `@ObservableObject` обёртку и используйте `.onReceive` или `.task`  
- **Для кастомных действий** — регистрируйте категории и кнопки через `UNNotificationCategory` заранее  
- **Privacy Manifest** (PrivacyInfo.xcprivacy) — обязательно с iOS 17+ для UserNotifications  
- **Документируйте** — пишите комментарий «UNUserNotificationCenterDelegate — обработка уведомлений на переднем плане и действий пользователя»

**Короткий итог 2026**:
> `UNUserNotificationCenterDelegate` — это **единственный** способ управлять поведением уведомлений в приложении.  
> В 2026 году:  
> - два главных метода: `willPresent` (уведомление при активном приложении) и `didReceive` (действие пользователя)  
> - всегда вызывайте `completionHandler`  
> - устанавливайте делегат один раз в [[AppDelegate]] / [[SceneDelegate]]  
> - это **обязательный** компонент для любого приложения с уведомлениями  
