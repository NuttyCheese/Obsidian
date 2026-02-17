#system_design
## Определение

**APNs (Apple Push Notification Service)** — это сервис Apple, который позволяет отправлять **push-уведомления** на [[iOS]], iPadOS, watchOS и macOS устройства.

> Push-уведомления позволяют **информировать пользователя о событиях, обновлениях или действиях** в приложении, даже если оно не активно.

---

## Основные компоненты архитектуры APNs

|Компонент|Роль|
|---|---|
|**Device (устройство пользователя)**|Получает push-уведомления, отображает их.|
|**App (клиентское приложение)**|Регистрируется в APNs, получает уникальный device token.|
|**Provider (сервер приложения)**|Отправляет уведомления в APNs с помощью device token.|
|**APNs (сервер Apple)**|Принимает уведомления от провайдера, доставляет их на устройства.|

---

## Workflow доставки уведомлений

1. **Регистрация устройства в APNs**
    
    - Приложение запрашивает разрешение на push.
        
    - iOS предоставляет **device token** для идентификации устройства.
        
2. **Отправка device token на сервер приложения**
    
    - Сервер хранит токен для отправки уведомлений конкретному устройству.
        
3. **Отправка уведомления через сервер APNs**
    
    - Сервер формирует payload и отправляет его в APNs по [[HTTPS]].
        
    - Payload может содержать текст, badge, sound и custom data.
        
4. **Доставка уведомления на устройство**
    
    - APNs маршрутизирует уведомление на устройство с указанным device token.
        
    - Уведомление отображается, либо запускает приложение в фоне (silent push).
        

---

## Пример работы с APNs в iOS (Swift)

### 1. Регистрация на push

```swift
import UIKit
import UserNotifications

UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .sound, .badge]) { granted, error in
    if granted {
        DispatchQueue.main.async {
            UIApplication.shared.registerForRemoteNotifications()
        }
    }
}
```

### 2. Получение device token

```swift
func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
    let tokenString = deviceToken.map { String(format: "%02.2hhx", $0) }.joined()
    print("Device token: \(tokenString)")
    // Отправить token на сервер
}
```

### 3. Обработка ошибок регистрации

```swift
func application(_ application: UIApplication, didFailToRegisterForRemoteNotificationsWithError error: Error) {
    print("Failed to register: \(error)")
}
```

---

## Формат payload push-уведомления

```json
{
  "aps": {
    "alert": {
      "title": "Новый заказ",
      "body": "Ваш заказ #123 готов к доставке"
    },
    "badge": 1,
    "sound": "default"
  },
  "customData": {
    "orderId": "123"
  }
}
```

**Ключевые поля `aps`:**

- `alert` → текст уведомления (title, body).
    
- `badge` → обновление значка на иконке приложения.
    
- `sound` → звук уведомления.
    
- `content-available: 1` → silent push для фонового обновления данных.
    

---

## Типы push-уведомлений

|Тип|Описание|
|---|---|
|**User-visible**|Показываются пользователю, могут содержать alert, sound, badge|
|**Silent push**|Не отображаются, используются для фоновой синхронизации данных (`content-available: 1`)|
|**VoIP push**|Для звонков, использует отдельный сертификат (CallKit)|
|**Mutable content**|Позволяет модифицировать уведомление на устройстве через Notification Service Extension|

---

## Best Practices для iOS

1. **Использовать минимальный payload** для ускорения доставки.
    
2. **Silent push** → только для фонового обновления данных, не для отображения уведомлений.
    
3. **Отправка токенов** → сохранять на сервере, обновлять при изменении.
    
4. **Сегментация пользователей** → отправлять уведомления только нужной аудитории.
    
5. **Notification Service Extension** → кастомизация контента перед отображением.
    
6. **Retry и feedback** → обрабатывать недоставленные уведомления и устаревшие токены.
    

---

## Безопасность и сертификаты

- Для работы с APNs требуется **APNs Authentication Key (.p8)** или сертификаты для Production и Development.
    
- HTTPS соединение между сервером и APNs обязательно.
    
- Device token уникален для **каждого приложения на устройстве**.
    

---

## Итог

- **APNs** — основной механизм доставки push-уведомлений в iOS.
    
- Позволяет информировать пользователя и запускать фоновые процессы через silent push.
    
- Архитектура состоит из **клиента, сервера приложения и APNs**.
    
- Важные элементы: **device token, payload, сертификаты, обработка уведомлений и background fetch**.
    

---
