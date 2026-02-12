## 1. Что такое `Date`

**`Date`** — это структура ([[struct]]), представляющая **конкретный момент времени**.

- Хранит дату и время как количество секунд с **«эпохи»** (1 января 2001 года GMT).
    
- Не содержит информации о календаре или часовом поясе — это «чистый» момент времени.
    
- Для отображения или вычислений используют [[Calendar]], [[DateFormatter]] и `TimeZone`.
    

---

## 2. Создание Date

### Текущий момент времени

```swift
let now = Date()
print(now)
```

### Конкретная дата через Calendar и [[DateComponents]]

```swift
var components = DateComponents()
components.year = 2025
components.month = 8
components.day = 27
components.hour = 12
components.minute = 30

let calendar = Calendar.current
if let date = calendar.date(from: components) {
    print(date)
}
```

### Через [[TimeInterval]] (секунды от эпохи)

```swift
let pastDate = Date(timeIntervalSince1970: 0) // 1 января 1970
let futureDate = Date(timeIntervalSinceNow: 3600) // через 1 час
```

---

## 3. Форматирование даты

Используем `DateFormatter`:

```swift
let formatter = DateFormatter()
formatter.dateStyle = .medium   // Aug 27, 2025
formatter.timeStyle = .short    // 12:30
let dateString = formatter.string(from: now)
print(dateString)
```

Можно настроить формат вручную:

```swift
formatter.dateFormat = "dd.MM.yyyy HH:mm"
print(formatter.string(from: now)) // 27.08.2025 12:30
```

---

## 4. Получение компонентов даты

```swift
let calendar = Calendar.current
let year = calendar.component(.year, from: now)
let month = calendar.component(.month, from: now)
let day = calendar.component(.day, from: now)
let hour = calendar.component(.hour, from: now)
let minute = calendar.component(.minute, from: now)

print("Сегодня: \(day).\(month).\(year) \(hour):\(minute)")
```

---

## 5. Работа с интервалами

### Разница между датами

```swift
let startDate = Date()
let endDate = startDate.addingTimeInterval(3600) // через 1 час

let interval = endDate.timeIntervalSince(startDate)
print("Прошло секунд: \(interval)") // 3600
```

### Добавление/вычитание

```swift
let tomorrow = Calendar.current.date(byAdding: .day, value: 1, to: now)!
let lastWeek = Calendar.current.date(byAdding: .day, value: -7, to: now)!
```

---

## 6. Сравнение дат

```swift
if now > lastWeek {
    print("Сегодня позже, чем неделя назад")
}

if now == now {
    print("Даты одинаковы")
}
```

---

## 7. Итог

- `Date` = момент времени без календаря и часового пояса.
    
- Для работы с **календарными компонентами** и форматированием → `Calendar` и `DateFormatter`.
    
- Для вычислений интервалов → `TimeInterval` и методы `addingTimeInterval` / `date(byAdding:)`.
    
- Можно сравнивать, добавлять и вычитать даты.
    

---
