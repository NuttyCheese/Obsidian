#unnotification-service-extension #push-notifications #rich-notifications #notification-service-extension #ios #swift #background-processing #mutable-content #media-attachments #custom-notifications 

---
(сервисное расширение уведомлений / Notification Service Extension)

**UNNotificationServiceExtension** — это специальное расширение приложения (app extension), которое позволяет **модифицировать содержимое** входящих push-уведомлений **прямо перед их отображением** пользователю.

Оно работает **в фоновом режиме** (background execution) и даёт возможность:

- добавить медиа-вложения (картинки, видео, аудио)
- изменить заголовок, текст, подзаголовок уведомления
- заменить звук, категорию, badge
- добавить кастомные данные или отформатировать текст
- полностью скрыть уведомление (если нужно)

Это ключевой инструмент для **rich push-уведомлений** (богатых уведомлений) в iOS начиная с iOS 10 (2016) и остаётся актуальным в 2026 году.

### Зачем нужен UNNotificationServiceExtension в 2026 году

| Сценарий                                      | Что позволяет сделать расширение                               | Без расширения               |
| --------------------------------------------- | -------------------------------------------------------------- | ---------------------------- |
| Показать фото отправителя в сообщении         | Скачать фото по [[URL]] и прикрепить как вложение              | Только текст                 |
| Добавить превью видео/аудио в уведомлении     | Загрузить thumbnail или короткий клип                          | Только текст                 |
| Отобразить количество непрочитанных сообщений | Динамически обновить badge и текст                             | Статический badge            |
| Персонализировать текст уведомления           | Заменить плейсхолдеры на имя пользователя, сумму заказа и т.д. | Статический текст            |
| Скрыть уведомление при определённых условиях  | Вернуть [[nil]] или пустое содержимое                          | Уведомление всегда покажется |
| Добавить кнопки действий (custom actions)     | Добавить category с действиями (ответить, лайкнуть и т.д.)     | Только стандартные действия  |

### Как работает Notification Service Extension

1. Приходит push с ключом `"mutable-content": 1` в payload
2. Система запускает ваше расширение (в фоне, ~30 секунд)
3. Расширение получает [[UNNotificationRequest]]
4. Вы модифицируете `content` (заголовок, тело, вложения и т.д.)
5. Вызываете `contentHandler(modifiedContent)` → уведомление показывается
6. Если время истекло или ошибка → вызывается `contentHandler(originalContent)`

**Важно:** расширение **не может** показывать уведомление, если в payload нет `"mutable-content": 1`.

### Структура проекта и настройка (2026 актуально)

1. **Добавьте расширение в проект**  
   File → New → Target → Notification Service Extension  
   Назовите, например, `NotificationService`

2. **Основной файл — NotificationService.swift**

```swift
import UserNotifications

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
        
        // 1. Получаем данные из push
        guard let userInfo = bestAttemptContent.userInfo as? [String: Any],
              let imageURLString = userInfo["image_url"] as? String,
              let imageURL = URL(string: imageURLString) else {
            contentHandler(bestAttemptContent)
            return
        }
        
        // 2. Скачиваем изображение (асинхронно)
        URLSession.shared.downloadTask(with: imageURL) { [weak self] location, response, error in
            guard let self,
                  let location,
                  error == nil,
                  let attachment = try? UNNotificationAttachment(identifier: "image",
                                                                 url: location,
                                                                 options: nil) else {
                contentHandler(bestAttemptContent)
                return
            }
            
            // 3. Добавляем вложение
            bestAttemptContent.attachments = [attachment]
            
            // 4. Можно изменить текст
            bestAttemptContent.title = "\(bestAttemptContent.title) 📸"
            
            // 5. Передаём модифицированный контент
            contentHandler(bestAttemptContent)
        }.resume()
    }
    
    // 6. Если время истекло — показываем оригинальное уведомление
    override func serviceExtensionTimeWillExpire() {
        if let contentHandler, let bestAttemptContent {
            contentHandler(bestAttemptContent)
        }
    }
}
```

### Payload push-уведомления (обязательно)

```json
{
    "aps": {
        "alert": {
            "title": "Новое сообщение",
            "body": "Привет! Посмотри фото"
        },
        "mutable-content": 1,          // ← обязательно!
        "sound": "default"
    },
    "image_url": "https://example.com/photo.jpg"
}
```

### Лучшие практики UNNotificationServiceExtension в 2026 году

- **Время жизни** — ~30 секунд (Apple может убить раньше при низком заряде)
- **Скачивание** — используйте [[URLSession]] с `downloadTask` (не dataTask — экономит память)
- **Размер вложений** — до 1 МБ на изображение (рекомендация Apple), иначе уведомление не покажется
- **Кэширование** — используйте [[URLCache]] или сохраняйте вложения в группу контейнера (App Group)
- **Обработка ошибок** — всегда вызывайте `contentHandler` даже при ошибке (иначе уведомление не покажется)
- **Тестирование** — используйте локальные уведомления или симуляцию push через консоль
- **Для SwiftUI** — расширение остаётся в [[UIKit]], но можно интегрировать с [[SwiftUI]] через hosting
- **Для iOS 18+** — поддержка Live Activities, но для обычных уведомлений всё то же самое

**Короткий итог 2026**:
> **UNNotificationServiceExtension** — расширение, которое **модифицирует push-уведомления перед показом** (добавляет картинки, меняет текст).  
> Junior: "Добавляет фото и форматирует уведомления".  
> Middle: требует `"mutable-content": 1`, работает ~30 сек, использует `contentHandler`.  
> Senior: скачивайте через `downloadTask`, кэшируйте, обрабатывайте timeout, лимит 1 МБ на вложение, тестируйте через симуляцию.  
