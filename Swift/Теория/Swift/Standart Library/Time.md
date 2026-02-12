## 1. Что такое Time в [[Swift]]

В Swift **нет отдельного типа `Time`**, как в некоторых языках.  
Для работы с временем и датами используются:

- **`Date`** — конкретный момент времени (дата + время)
    
- **`TimeInterval`** — промежуток времени в секундах ([[Double]])
    
- **`Calendar` и `DateComponents`** — для работы с компонентами даты и времени
    

> Проще: если нужно «время» отдельно, обычно используем [[Date]] и форматируем только часы и минуты.

---

## 2. Получение текущего времени

```swift
let now = Date()
print(now) // 2025-08-27 12:34:56 +0000
```

---

## 3. Форматирование времени

Для отображения только времени используют [[DateFormatter]]:

```swift
let formatter = DateFormatter()
formatter.timeStyle = .short   // короткий формат: 12:34
formatter.dateStyle = .none    // без даты
let timeString = formatter.string(from: Date())
print("Сейчас время: \(timeString)")
```

Варианты `timeStyle`:

- `.short` → 12:34
    
- `.medium` → 12:34:56
    
- `.long` → 12:34:56 GMT+0
    
- `.full` → 12:34:56 GMT+0 Monday, 27 August 2025
    

---

## 4. Работа с компонентами времени

```swift
let calendar = Calendar.current
let now = Date()
let hour = calendar.component(.hour, from: now)
let minute = calendar.component(.minute, from: now)
let second = calendar.component(.second, from: now)

print("Часы: \(hour), Минуты: \(minute), Секунды: \(second)")
```

Можно создавать время отдельно через `DateComponents`:

```swift
var components = DateComponents()
components.hour = 14
components.minute = 30
components.second = 0

let calendar = Calendar.current
if let date = calendar.date(from: components) {
    print("Созданное время: \(date)")
}
```

---

## 5. [[TimeInterval]] — измерение промежутков времени

```swift
let start = Date()
// какой-то код
let end = Date()
let duration: TimeInterval = end.timeIntervalSince(start)
print("Прошло секунд: \(duration)")
```

- `TimeInterval` = Double, в секундах
    
- Можно конвертировать:
    

```swift
let seconds = duration
let minutes = duration / 60
let hours = duration / 3600
```

---

## 6. Итог

- В Swift **нет отдельного типа Time**, используют `Date`, `TimeInterval` и [[Calendar]].
    
- Для отображения только времени → `DateFormatter` с `timeStyle`.
    
- Для вычислений → `TimeInterval`.
    
- Для создания времени → [[DateComponents]].
    

---
