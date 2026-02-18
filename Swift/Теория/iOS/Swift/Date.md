**`Date`** в Swift — это **структура** (value type), представляющая **конкретный момент времени** — точку на временной шкале, независимо от календаря, формата или часового пояса.

Она хранит **время в секундах с эпохи** (1 января 2001 года 00:00:00 GMT — это точка отсчёта для `Date` в Swift и Foundation).

### 1. Самые важные факты о Date (2026 актуально)

| Характеристика                        | Значение / Особенность                                      | Важные детали 2026 |
|---------------------------------------|-------------------------------------------------------------|---------------------|
| Тип                                   | `struct` (value type, копируется)                           | Copy-on-Write не применяется (маленький размер) |
| Эпоха                                 | 1 января 2001 00:00:00 GMT                                  | Отличается от Unix-эпохи (1970) |
| Внутреннее представление              | `Double` — секунды с 2001 года                              | Отрицательные значения = до 2001 года |
| Размер в памяти                       | 8 байт (Double)                                             | Очень компактный |
| Не содержит                           | Календарь, часовой пояс, локаль, формат                     | Всё это — в `Calendar`, `TimeZone`, `DateFormatter` |
| Thread-safety                         | Полностью потокобезопасен (immutable по своей природе)      | Можно передавать между акторами без проблем |
| Основные операции                     | Сравнение, добавление/вычитание интервалов, компоненты через `Calendar` | — |

### 2. Самые популярные способы создания Date (2026)

```swift
// 1. Текущий момент (самый частый)
let now = Date()

// 2. Через TimeInterval (секунды от эпохи Swift — 2001)
let referenceDate = Date(timeIntervalSinceReferenceDate: 0)  // 1 января 2001
let unixEpoch = Date(timeIntervalSince1970: 0)               // 1 января 1970
let future = Date(timeIntervalSinceNow: 3600)                // через 1 час
let past = Date(timeIntervalSinceNow: -86400)                // сутки назад

// 3. Через DateComponents + Calendar (самый читаемый и безопасный)
var components = DateComponents()
components.year   = 2025
components.month  = 12
components.day    = 31
components.hour   = 23
components.minute = 59
components.second = 59

if let newYearEve = Calendar.current.date(from: components) {
    print("Новый год 2026:", newYearEve)
}

// 4. Через ISO8601 строку (очень частый в API)
let isoString = "2025-08-27T14:30:00Z"
if let date = ISO8601DateFormatter().date(from: isoString) {
    // ...
}
```

### 3. Самые полезные операции с Date

#### Сравнение дат

```swift
let now = Date()
let tomorrow = Calendar.current.date(byAdding: .day, value: 1, to: now)!

print(now < tomorrow)         // true
print(now == now)             // true
print(now.distance(to: tomorrow))  // ≈ 86400.0 (секунд)
```

#### Добавление/вычитание интервалов

```swift
let oneHourLater = now.addingTimeInterval(3600)
let oneWeekAgo   = now.addingTimeInterval(-7 * 86400)

// Более читаемо и безопасно через Calendar
let nextMonth = Calendar.current.date(byAdding: .month, value: 1, to: now)
let threeYearsAgo = Calendar.current.date(byAdding: .year, value: -3, to: now)
```

#### Разница между датами (самый частый вопрос)

```swift
let start = Date()
let end = Calendar.current.date(byAdding: .day, value: 10, to: start)!

let diff = Calendar.current.dateComponents([.day, .hour, .minute], from: start, to: end)
print("Прошло: \(diff.day ?? 0) дней, \(diff.hour ?? 0) часов")

// Только полные дни (игнорируя время)
let days = Calendar.current.dateComponents([.day], from: start, to: end).day ?? 0
```

#### Получение компонентов (через Calendar)

```swift
let cal = Calendar.current
let year   = cal.component(.year,   from: now)
let month  = cal.component(.month,  from: now)
let day    = cal.component(.day,    from: now)
let weekday = cal.component(.weekday, from: now) // 1 = воскресенье (зависит от локали!)
```

### 4. Лучшие практики Date в Swift 2026

- **Никогда** не храните `Date` в виде строки вручную — используйте `ISO8601DateFormatter` или `Codable`  
- **Всегда** используйте `Calendar.current` — он учитывает локаль и пояс пользователя  
- **Для хранения дат** — используйте `Date` напрямую (в Core Data, UserDefaults, JSON)  
- **Для отображения** — используйте `DateFormatter` с `dateStyle`/`timeStyle` или `formatted()` (iOS 15+)  
- **Для вычислений** — предпочитайте `Calendar.date(byAdding:...)` над `addingTimeInterval` — учитывает високосные годы и переходы DST  
- **Для временных интервалов** — используйте `DateInterval` (iOS 10+)  
- **Swift 6 strict concurrency** — `Date` полностью `Sendable` и безопасен  
- **Документируйте** — пиши комментарий «Date — момент создания записи (UTC)»

**Короткий девиз 2026**:
> `Date` — это **чистый момент времени** без календаря и пояса.  
> В 2026 году:  
> - создавай через `Calendar.date(from:)` или `Date()`  
> - считай разницу через `Calendar.dateComponents`  
> - добавляй/вычитай через `Calendar.date(byAdding:...)`  
> - форматируй через `DateFormatter` или `.formatted()`  
> Это **единственный правильный** способ работать с датами в Swift.

Удачи с точными и локализованными датами в твоём приложении! 📅