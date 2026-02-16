**APNs** (Apple Push Notification service) — это сервис Apple для отправки **push-уведомлений** на устройства iOS, iPadOS, macOS, watchOS, tvOS и visionOS.

По состоянию на февраль 2026 года APNs остаётся **единственным официальным способом** доставки push-уведомлений на устройства Apple.

### Основные изменения и актуальное состояние APNs (2026)

| Характеристика                  | Статус на 2026 год                                                                 | Что важно знать |
|---------------------------------|-------------------------------------------------------------------------------------|-----------------|
| **HTTP/2 API**                  | Основной и единственный поддерживаемый протокол (с 2016 года)                      | APNs больше не поддерживает бинарный протокол |
| **Token-based authentication**  | **Обязательна** с 2021 года (JWT-токены)                                           | Сертификаты (.p12) deprecated с 2025 года |
| **VoIP push**                   | Полностью поддерживается, но требует специального entitlement                     | Для звонков и VoIP-приложений |
| **Live Activities**             | Полная поддержка (с iOS 16.1+, 2022–2026 активно развивается)                     | Динамические уведомления на Lock Screen |
| **Push to Start** (iOS 18+)     | Поддержка уведомлений для запуска приложения по событию                           | Актуально для iOS 18+ |
| **Rich push** (изображения, кнопки, категории) | Полная поддержка (с iOS 10+)                                                      | Максимум 4 кнопки, 1 изображение |
| **Critical Alerts**             | Поддержка (с iOS 12+), требует специального entitlement от Apple                  | Работают даже при беззвучном режиме |
| **Alert / Banner / Sound / Badge** | Всё работает как раньше                                                            | Стандартные типы уведомлений |
| **Максимальная длина payload**  | 4 КБ (4096 байт) для обычных push, 5 КБ для VoIP                                  | JSON-структура |
| **Частота отправки**            | Нет жёсткого лимита, но Apple может троттлить при злоупотреблении                | ~1–2 млн/сек глобально (по данным Apple WWDC) |

### Современная архитектура push в iOS-приложении 2026

```text
Frontend (SwiftUI / UIKit)
      ↓ (регистрируется на APNs)
Device Token → AppDelegate / SceneDelegate / UNUserNotificationCenter
      ↓ (отправляется на ваш бэкенд)
Ваш сервер (Node.js / Python / Go / Vapor / Firebase Functions)
      ↓ (генерирует JWT-токен)
APNs (api.push.apple.com:443)
      ↓ (доставляет уведомление)
Устройство (iOS 18+ / macOS 15+)
      ↓ (показывает баннер / звук / badge / Live Activity)
```

### Самый актуальный пример отправки push через APNs (2026)

#### Шаг 1. Получение Device Token в приложении

```swift
import UserNotifications

class AppDelegate: UIResponder, UIApplicationDelegate, UNUserNotificationCenterDelegate {
    
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        UNUserNotificationCenter.current().delegate = self
        
        UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .sound, .badge]) { granted, error in
            if granted {
                DispatchQueue.main.async {
                    application.registerForRemoteNotifications()
                }
            }
        }
        return true
    }
    
    func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        let token = deviceToken.map { String(format: "%02.2hhx", $0) }.joined()
        print("Device Token: \(token)")
        
        // Отправьте token на ваш бэкенд
        sendTokenToBackend(token)
    }
    
    func application(_ application: UIApplication, didFailToRegisterForRemoteNotificationsWithError error: Error) {
        print("Failed to register: \(error)")
    }
}
```

#### Шаг 2. Отправка push с бэкенда (пример на Swift + Vapor / Node.js / Python)

**Swift (Vapor 4+)**

```swift
import Vapor
import JWTKit

struct APNsPayload: Content {
    let aps: Aps
}

struct Aps: Content {
    let alert: Alert?
    let badge: Int?
    let sound: String?
    let category: String?
}

struct Alert: Content {
    let title: String?
    let body: String?
}

func sendPush(to token: String, title: String, body: String) async throws {
    let signer = JWTSigner.hs256(key: "YOUR_AUTH_KEY".data(using: .utf8)!)
    
    let headers = HTTPHeaders([
        ("apns-push-type", "alert"),
        ("apns-priority", "10"),
        ("apns-topic", "com.your.bundle.id")
    ])
    
    let payload = APNsPayload(
        aps: Aps(
            alert: Alert(title: title, body: body),
            badge: 1,
            sound: "default",
            category: nil
        )
    )
    
    let jwt = try signer.sign(APNsJWT(iss: "YOUR_TEAM_ID", iat: Date()))
    
    let client = try await req.client
    _ = try await client.post(
        "https://api.push.apple.com/3/device/\(token)",
        headers: headers + [("authorization", "Bearer \(jwt)")]
    ) { req in
        try req.content.encode(payload)
    }
}
```

**Python (пример с pyapns2 / apns2)**

```python
from apns2.client import APNsClient
from apns2.payload import Payload

client = APNsClient(
    credentials=TokenCredentials(
        auth_key_path="AuthKey_XXXXXXXXXX.p8",
        key_id="XXXXXXXXXX",
        team_id="XXXXXXXXXX"
    ),
    use_sandbox=False
)

payload = Payload(alert=PayloadAlert(title="Hello", body="This is a test"), badge=1, sound="default")

client.send_notification(
    device_token="device_token_here",
    payload=payload,
    topic="com.your.bundle.id"
)
```

### Лучшие практики APNs в 2026 году

- **Только token-based auth** — сертификаты .p12 полностью отключены с 2025 года  
- **apns-push-type** — обязательно указывать: `alert`, `background`, `voip`, `liveactivity`, `push-to-start`  
- **apns-priority** — 10 для видимых уведомлений, 5 для фоновых  
- **apns-collapse-id** — для группировки уведомлений (чтобы старые заменялись новыми)  
- **apns-topic** — всегда ваш bundle ID  
- **Live Activities** — используйте `push-to-start` и `liveactivity` типы для динамических уведомлений  
- **Critical Alerts** — требуют специального entitlement от Apple (очень сложно получить)  
- **Тестирование** — используйте **Sandbox** (api.sandbox.push.apple.com) + реальное устройство  
- **Отладка** — смотрите логи в Console.app → фильтр по `apnsd`  
- **Мониторинг** — используйте метрики APNs (через ваш бэкенд) или сторонние сервисы (OneSignal, Firebase, Braze)

**Короткий девиз 2026**:
> «APNs в 2026 году — это когда ты хочешь надёжно доставить уведомление пользователю.  
> Единственный правильный способ — token-based JWT + HTTP/2 API.  
> Всё остальное — legacy или нарушение правил Apple.»

Удачи с надёжными push-уведомлениями в вашем iOS-приложении! 📬