#unnotification-content-extension #rich-notifications #custom-ui #notification-content-extension #ios #swift #user-notifications #media-attachments #interactive-notifications #ios-10

---
(расширение содержимого уведомлений / Notification Content Extension)

**UNNotificationContentExtension** — это **расширение приложения** (app extension), которое позволяет **полностью заменить стандартный интерфейс** уведомления на **кастомный UI**, созданный вами.

Оно появилось в **iOS 10** (2016) и до сих пор (2026 год) остаётся единственным официальным способом сделать **по-настоящему богатые и интерактивные уведомления** с:

- кастомным layout
- изображениями / видео / аудио / анимациями
- кнопками действий внутри уведомления
- интерактивными элементами (слайдеры, переключатели, текст-поля)
- динамическим обновлением содержимого

Это **второй уровень** богатых уведомлений после **[[UNNotificationServiceExtension]]** (модификация контента).  
Обычно их используют вместе: Service Extension → добавляет медиа, Content Extension → показывает красивый UI с этим медиа.

### Когда использовать UNNotificationContentExtension в 2026 году

| Сценарий                                      | Что позволяет сделать расширение                                   | Без расширения |
|-----------------------------------------------|---------------------------------------------------------------------|----------------|
| Уведомление о новом сообщении в чате          | Показать аватар, имя, текст + кнопки «Ответить», «Лайк»            | Только текст + стандартные действия |
| Уведомление о доставке заказа                 | Карта с местоположением курьера + кнопка «Отследить»               | Только текст |
| Уведомление о новом эпизоде подкаста          | Обложка альбома + кнопки Play/Pause + прогресс                      | Только текст |
| Интерактивное уведомление о будильнике        | Слайдер «Отложить на 10 мин» + кнопка «Выключить»                  | Только кнопки |
| Уведомление о погоде / спорте                  | Анимированная иконка погоды / счёт матча + кнопки действий         | Статический текст |
| Уведомление с видео-превью                    | Автовоспроизведение короткого видео (mute)                          | Только статичное изображение |

### Как работает Notification Content Extension

1. Пользователь получает push с ключом `"category"` в payload
2. Система показывает уведомление
3. При раскрытии (3D Touch / long press / swipe down) вызывается ваше расширение
4. Расширение загружает кастомный UI из своего storyboard / [[SwiftUI]] Hosting
5. Пользователь взаимодействует с вашим UI (кнопки, слайдеры и т.д.)
6. Действия передаются в приложение через [[UNNotificationResponse]]

**Важно:** расширение **не заменяет** уведомление полностью — оно только добавляет **расширенный вид** при раскрытии.

### Настройка проекта (2026 актуально)

1. **Добавьте расширение**  
   File → New → Target → Notification Content Extension  
   Назовите, например, `NotificationContent`

2. **Выберите категорию** в Info.plist расширения

```xml
<key>NSExtension</key>
<dict>
    <key>NSExtensionAttributes</key>
    <dict>
        <key>UNNotificationExtensionCategory</key>
        <string>chat-message-category</string>     <!-- ваша категория -->
        <key>UNNotificationExtensionDefaultContentHidden</key>
        <true/>                                     <!-- скрыть стандартный текст -->
        <key>UNNotificationExtensionInitialContentSizeRatio</key>
        <real>0.7</real>                            <!-- соотношение высоты -->
    </dict>
    <key>NSExtensionMainStoryboard</key>
    <string>NotificationViewController</string>
</dict>
```

3. **Основной контроллер — NotificationViewController.swift**

```swift
import UIKit
import UserNotifications
import UserNotificationsUI

class NotificationViewController: UIViewController, UNNotificationContentExtension {
    
    @IBOutlet weak var titleLabel: UILabel!
    @IBOutlet weak var bodyLabel: UILabel!
    @IBOutlet weak var imageView: UIImageView!
    @IBOutlet weak var playButton: UIButton!
    
    override func viewDidLoad() {
        super.viewDidLoad()
    }
    
    // Самый важный метод — система передаёт нам содержимое уведомления
    func didReceive(_ notification: UNNotification) {
        let content = notification.request.content
        
        titleLabel.text = content.title
        bodyLabel.text = content.body
        
        // Если есть вложения (добавлены через Service Extension)
        if let attachment = content.attachments.first {
            if attachment.url.startAccessingSecurityScopedResource() {
                if let imageData = try? Data(contentsOf: attachment.url),
                   let image = UIImage(data: imageData) {
                    imageView.image = image
                }
                attachment.url.stopAccessingSecurityScopedResource()
            }
        }
    }
    
    // Реакция на нажатие кнопок в уведомлении
    @IBAction func playButtonTapped(_ sender: UIButton) {
        // Можно отправить действие в приложение
        // или воспроизвести звук/видео прямо в уведомлении
    }
    
    // Опционально: динамически менять размер уведомления
    func didReceive(_ response: UNNotificationResponse, completionHandler completion: @escaping (UNNotificationContentExtensionResponseOption) -> Void) {
        // Пользователь нажал на кастомную кнопку
        if response.actionIdentifier == "play-action" {
            // Воспроизвести аудио прямо в уведомлении
            completion(.doNotDismiss)  // не закрывать уведомление
        } else {
            completion(.dismissAndForwardAction)  // закрыть и передать действие в приложение
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
            "body": "Привет! Как дела?"
        },
        "category": "chat-message-category",   // ← совпадает с Info.plist
        "mutable-content": 1,                   // для Service Extension
        "sound": "default"
    },
    "attachment-url": "https://example.com/photo.jpg"
}
```

### Лучшие практики UNNotificationContentExtension в 2026 году

- **Размер уведомления** — не больше 1/3 экрана (iPhone) или 1/2 (iPad)
- **Медиа** — добавляйте через Service Extension заранее (вложение до 1 МБ)
- **Автовоспроизведение** — видео/аудио в уведомлении — только mute и короткие (до 30 сек)
- **Кнопки действий** — регистрируйте категорию через [[UNUserNotificationCenter]] в приложении
- **Производительность** — расширение живёт очень мало времени → загружайте минимум данных
- **Тестирование** — используйте локальные уведомления + симуляцию push через консоль
- **Для [[SwiftUI]]** — можно использовать [[UIHostingController]] внутри расширения
- **Для iOS 18+** — поддержка Live Activities, но Content Extension остаётся для классических уведомлений

**Короткий итог 2026**:
> **UNNotificationContentExtension** — расширение, которое **заменяет стандартный UI уведомления** на ваш кастомный при раскрытии.  
> Junior: "Делает уведомления красивыми с картинками и кнопками".  
> Middle: требует категории + mutable-content, работает только при раскрытии, ~30 сек жизни.  
> Senior: комбинируйте с Service Extension, кэшируйте медиа, используйте `didReceive(response:)` для интерактивности, лимит размера.  
