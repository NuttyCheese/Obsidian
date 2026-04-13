#unnotificationtrigger #usernotifications #local-notifications #scheduled-notifications #push-notifications #ios #swift #triggers #timeinterval #calendar #location #ios-10

---
(триггер уведомления / условие срабатывания уведомления)

**UNNotificationTrigger** — это **абстрактный базовый класс** в фреймворке UserNotifications (с iOS 10+), который определяет **условие**, при котором система должна показать уведомление.

Без триггера уведомление **никогда не покажется** автоматически (кроме немедленного показа через `UNUserNotificationCenter.add(_:withCompletionHandler:)` с `trigger = nil`).

### Основные подклассы UNNotificationTrigger (2026 актуально)

| Подкласс                              | Когда срабатывает                                   | Повторяющийся? | Самый частый сценарий 2026                       |
| ------------------------------------- | --------------------------------------------------- | -------------- | ------------------------------------------------ |
| `UNTimeIntervalNotificationTrigger`   | Через заданное количество секунд после планирования | Да (repeats)   | Напоминания через 5 мин, 1 час                   |
| `UNCalendarNotificationTrigger`       | В определённое время/день/месяц/год по календарю    | Да             | Ежедневный будильник, еженедельные события       |
| `UNLocationNotificationTrigger`       | При входе/выходе из геозоны (координаты + радиус)   | Да             | "Добро пожаловать домой", "Вы рядом с магазином" |
| `UNPushNotificationTrigger` (неявный) | При получении push от APNs (trigger = [[nil]])      | —              | Все удалённые уведомления                        |
| `UNNotificationTrigger` (абстрактный) | Базовый класс — не создаётся напрямую               | —              | —                                                |

### 1. UNTimeIntervalNotificationTrigger  
(уведомление через интервал времени)

Самый простой и популярный для напоминаний.

```swift
// Через 60 секунд, без повтора
let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 60, repeats: false)

// Каждый день в 9:00 (через 24 часа)
let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 86400, repeats: true)

// Важно: минимальный интервал для повтора — 60 секунд
```

### 2. UNCalendarNotificationTrigger  
(уведомление по календарю)

```swift
var dateComponents = DateComponents()
dateComponents.hour = 20
dateComponents.minute = 0

// Каждый день в 20:00
let trigger = UNCalendarNotificationTrigger(dateMatching: dateComponents, repeats: true)

// Конкретная дата (например, 1 января 2027 в 10:00)
var specific = DateComponents()
specific.year = 2027
specific.month = 1
specific.day = 1
specific.hour = 10
specific.minute = 0

let oneTimeTrigger = UNCalendarNotificationTrigger(dateMatching: specific, repeats: false)
```

### 3. UNLocationNotificationTrigger  
(гео-уведомления)

```swift
let center = CLLocationCoordinate2D(latitude: 55.7558, longitude: 37.6173) // Москва
let region = CLCircularRegion(center: center, radius: 500, identifier: "moscow-center")

// Срабатывает при входе в радиус 500 метров
region.notifyOnEntry = true
region.notifyOnExit = false

let trigger = UNLocationNotificationTrigger(region: region, repeats: true)
```

**Важно:**
- Требуется разрешение `CLLocationManager().requestAlwaysAuthorization()`
- Радиус минимум 100 метров (Apple может игнорировать меньше)
- Работает в фоне (при `always` разрешении)

### Полный пример планирования уведомления (2026 стиль)

```swift
import UserNotifications

func scheduleDailyWaterReminder() {
    let center = UNUserNotificationCenter.current()
    
    // Запрос разрешения (обычно в AppDelegate)
    center.requestAuthorization(options: [.alert, .sound, .badge]) { granted, _ in
        guard granted else { return }
    }
    
    // Содержимое
    let content = UNMutableNotificationContent()
    content.title = "Время пить воду 💧"
    content.body = "Ты уже давно не пил! Сделай глоток прямо сейчас"
    content.sound = .default
    content.badge = 1
    content.categoryIdentifier = "hydration-reminder"
    
    // Триггер — каждый день в 20:00
    var components = DateComponents()
    components.hour = 20
    components.minute = 0
    
    let trigger = UNCalendarNotificationTrigger(dateMatching: components, repeats: true)
    
    // Запрос
    let request = UNNotificationRequest(
        identifier: "daily-water-\(UUID().uuidString)",
        content: content,
        trigger: trigger
    )
    
    center.add(request) { error in
        if let error {
            print("Ошибка планирования: \(error)")
        } else {
            print("Напоминание запланировано")
        }
    }
}
```

### Лучшие практики UNNotificationTrigger в 2026 году

- **identifier** — всегда уникальный ([[UUID]] или server-side ID для remote)
- **repeats = true** — только если интервал ≥ 60 секунд (иначе игнорируется)
- **Календарный триггер** — используйте `repeats: true` для ежедневных/еженедельных событий
- **Гео-триггер** — обязательно `CLLocationManager` + `always` разрешение + обработка `didEnterRegion`
- **Тестирование** — для локальных: `center.add(request)`; для remote — симуляция push через консоль
- **Ограничения** — до 64 локальных уведомлений одновременно (остальные игнорируются)
- **Для iOS 18+** — поддержка Live Activities, но триггеры остаются теми же
- **Для SwiftUI** — всё через UserNotifications, но UI можно сделать в Content Extension

**Короткий итог 2026**:
> **UNNotificationTrigger** — условие, **когда показать уведомление** (время, календарь, локация, push).  
> Junior: "Говорит системе, когда выдать уведомление".  
> Middle: три подкласса — TimeInterval, Calendar, Location; repeats только ≥60 сек.  
> Senior: используйте уникальные identifier, комбинируйте с категориями и Service/Content Extensions, тестируйте локально, учитывайте лимит 64 уведомления.  
