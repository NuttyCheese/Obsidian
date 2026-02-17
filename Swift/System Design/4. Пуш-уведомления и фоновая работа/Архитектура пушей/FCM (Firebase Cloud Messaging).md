#system_design
## Определение

**Firebase Cloud Messaging (FCM)** — это сервис Google, который позволяет **отправлять push-уведомления и данные на мобильные приложения и веб-клиенты**.

- Поддерживает [[iOS]], Android и веб.
    
- Используется для уведомлений и фоновой синхронизации.
    

> FCM удобен для кроссплатформенных приложений, так как объединяет управление уведомлениями и отправку данных через единый API.

---

## Основные компоненты архитектуры FCM

|Компонент|Роль|
|---|---|
|**Client App (iOS/Android/Web)**|Получает push-уведомления и данные, регистрируется в FCM.|
|**Firebase Server / Admin SDK**|Отправляет уведомления и данные в FCM для доставки клиентам.|
|**FCM Server (Google)**|Маршрутизирует уведомления на устройства, обеспечивает доставку.|
|**Device Token / Registration Token**|Уникальный идентификатор устройства для FCM.|

---

## Workflow доставки уведомлений

1. **Регистрация приложения в FCM**
    
    - Получение **FCM token** для идентификации устройства.
        
    - iOS приложение использует **APNs** под капотом, FCM автоматически управляет взаимодействием.
        
2. **Отправка токена на сервер приложения**
    
    - Сервер хранит токен для адресной доставки уведомлений.
        
3. **Отправка уведомления через FCM**
    
    - Используется **HTTP v1 API** или Admin SDK.
        
    - Payload может содержать текст, badge, sound, data.
        
4. **Доставка уведомления на устройство**
    
    - FCM взаимодействует с **APNs** для iOS устройств.
        
    - На Android доставка осуществляется напрямую через FCM.
        

---

## Типы сообщений FCM

|Тип|Описание|
|---|---|
|**Notification message**|Автоматически отображается пользователю (title, body, sound).|
|**Data message**|Передаёт данные для обработки приложением (silent push), не отображается напрямую.|
|**Combined message**|И notification, и data одновременно, можно обработать custom logic.|

---

## Пример интеграции FCM в iOS (Swift)

### 1. Подключение [[Firebase]]

- Через [[SPM]] или [[CocoaPods]]:
    

```ruby
pod 'Firebase/Messaging'
```

### 2. Настройка AppDelegate

```swift
import Firebase
import UserNotifications

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate, UNUserNotificationCenterDelegate, MessagingDelegate {

    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        FirebaseApp.configure()

        UNUserNotificationCenter.current().delegate = self
        UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .badge, .sound]) { granted, error in
            if granted {
                DispatchQueue.main.async {
                    application.registerForRemoteNotifications()
                }
            }
        }

        Messaging.messaging().delegate = self
        return true
    }

    func messaging(_ messaging: Messaging, didReceiveRegistrationToken fcmToken: String?) {
        print("FCM token: \(fcmToken ?? "")")
        // Отправить токен на сервер
    }
}
```

---

## Payload FCM (пример)

```json
{
  "to": "DEVICE_FCM_TOKEN",
  "notification": {
    "title": "Новый заказ",
    "body": "Ваш заказ #123 готов к доставке",
    "sound": "default"
  },
  "data": {
    "orderId": "123",
    "status": "ready"
  }
}
```

- `notification` → отображение пользователю.
    
- `data` → передача данных для обработки приложением (background updates).
    

---

## Silent push через FCM

- Для обновления данных без показа уведомления:
    

```json
{
  "to": "DEVICE_FCM_TOKEN",
  "content_available": true,
  "priority": "high",
  "data": {
    "updateType": "syncOrders"
  }
}
```

- Приложение получит событие в фоне через `application(_:didReceiveRemoteNotification:fetchCompletionHandler:)`.
    

---

## Best Practices

1. **Использовать высокую приоритетность для важных уведомлений** (`priority: "high"`).
    
2. **Silent push** → для фоновой синхронизации, обновления локального кэша.
    
3. **Обновление FCM токенов** → токен может изменяться, нужно обновлять на сервере.
    
4. **Сегментация пользователей** → отправка только целевой аудитории через топики.
    
5. **Объединение notification и data** → гибкость для отображения и обработки.
    

---

## Преимущества FCM для iOS

- Кроссплатформенная доставка уведомлений.
    
- Автоматическая работа с APNs для iOS.
    
- Поддержка silent push и data messages.
    
- Легкая интеграция через Admin SDK и [[Swift/Теория/Сетевое взаимодействие/REST API]].
    
- Возможность таргетинга и сегментации пользователей.
    

---

## Итог

- **FCM** — современный сервис Google для push-уведомлений и фоновой синхронизации.
    
- Для iOS использует **APNs под капотом**, поэтому интеграция упрощает работу с уведомлениями.
    
- Позволяет отправлять **user-visible notifications**, **silent push** и **data messages**.
    
- Ключевые элементы: **FCM token, notification/data payload, сегментация пользователей, обработка в фоне**.
    

---
