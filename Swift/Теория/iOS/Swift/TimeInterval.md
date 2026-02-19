**`TimeInterval`** — это **псевдоним** ([[typealias]]) для типа [[Double]], который используется в [[Swift]] исключительно для представления **продолжительности времени в секундах** (с долями секунды).

```swift
typealias TimeInterval = Double
```

Это **не отдельный тип**, а просто удобное название, чтобы код был читаемым и семантически понятным.

### Основные сценарии использования TimeInterval (2025–2026)

| Задача                            | Как обычно выглядит в коде                                         | Примерное значение |
| --------------------------------- | ------------------------------------------------------------------ | ------------------ |
| Разница между двумя датами        | `end.timeIntervalSince(start)`                                     | 125.7 сек          |
| Добавить/вычесть время к [[Date]] | `date.addingTimeInterval(3600)`                                    | +1 час             |
| Интервал для [[Timer]]            | `Timer.scheduledTimer(withTimeInterval: 0.5, repeats: true) { … }` | 0.5 сек            |
| Длительность анимации             | `UIView.animate(withDuration: 0.3) { … }`                          | 0.3 сек            |
| Задержка в [[DispatchQueue]]      | `DispatchQueue.main.asyncAfter(deadline: .now() + 2.5) { … }`      | 2.5 сек            |
| Таймаут сетевого запроса          | `URLSessionConfiguration.default.timeoutIntervalForRequest = 30`   | 30 сек             |

### Самые употребительные идиомы и лучшие практики 2026 года

#### 1. Разница между двумя моментами времени

```swift
let start = Date()
// ... какая-то операция
let end = Date()

let duration: TimeInterval = end.timeIntervalSince(start)

print(String(format: "Затрачено: %.2f сек", duration))
// или более красиво:
let formatter = DateComponentsFormatter()
formatter.allowedUnits = [.minute, .second]
formatter.unitsStyle = .abbreviated
print(formatter.string(from: duration) ?? "—")   // "0 мин 2.34 сек"
```

#### 2. Самая читаемая конвертация секунд в ч:мм:сс

```swift
extension TimeInterval {
    var formattedHMS: String {
        let totalSeconds = Int(self)
        let hours = totalSeconds / 3600
        let minutes = (totalSeconds % 3600) / 60
        let seconds = totalSeconds % 60
        
        if hours > 0 {
            return String(format: "%dч %02dм %02dс", hours, minutes, seconds)
        } else if minutes > 0 {
            return String(format: "%02dм %02dс", minutes, seconds)
        } else {
            return String(format: "%02dс", seconds)
        }
    }
}

let duration: TimeInterval = 3665.7
print(duration.formattedHMS)   // "1ч 01м 05с"
```

#### 3. Добавление/вычитание интервала к дате (очень часто)

```swift
let now = Date()

let in30Minutes   = now.addingTimeInterval(30 * 60)
let fiveDaysAgo   = now.addingTimeInterval(-5 * 24 * 3600)
let twoHoursLater = now + 2 * 3600          // оператор + перегружен
```

#### 4. Таймер с повтором (самый частый паттерн)

```swift
var timer: Timer?

func startPolling() {
    timer = Timer.scheduledTimer(withTimeInterval: 5.0, repeats: true) { _ in
        fetchLatestData()
    }
}

func stopPolling() {
    timer?.invalidate()
    timer = nil
}
```

#### 5. Анимация с разной длительностью

```swift
let duration: TimeInterval = isFastMode ? 0.25 : 0.6

UIView.animate(withDuration: duration, delay: 0, options: .curveEaseOut) {
    self.view.alpha = 0
}
```

### 6. Полезные расширения (очень популярны в 2026)

```swift
extension TimeInterval {
    var seconds:   Double { self }
    var minutes:   Double { self / 60 }
    var hours:     Double { self / 3600 }
    var days:      Double { self / 86_400 }
    
    static var oneSecond:   TimeInterval { 1 }
    static var oneMinute:   TimeInterval { 60 }
    static var fiveMinutes: TimeInterval { 300 }
    static var oneHour:     TimeInterval { 3600 }
    static var oneDay:      TimeInterval { 86_400 }
}
```

Использование:

```swift
Timer.scheduledTimer(withTimeInterval: .fiveMinutes, repeats: true) { … }
let timeout = 30.seconds
```

### 7. Лучшие практики TimeInterval в 2026

- **Всегда** используйте **именованные константы** или расширения — `30 * 60` хуже, чем `.thirtyMinutes` или `30.minutes`  
- **Для человекочитаемого вывода** — `DateComponentsFormatter` (не пишите велосипед с % и /)  
- **Для точности** — помните, что `TimeInterval` — это `Double`, поэтому возможны погрешности при очень долгих интервалах  
- **Не путайте** с `DateComponents` — `TimeInterval` — это просто секунды, без календарных понятий  
- **В SwiftUI** — чаще используйте `.task { try await Task.sleep(for: .seconds(2)) }` вместо `Timer`  
- **Документируйте** — пишите комментарий «TimeInterval — таймаут запроса в секундах (30 сек)»

**Короткий итог 2026**:
> `TimeInterval` — это просто `Double`, но **с семантикой секунд**.  
> Используйте его для:  
> - разницы между `Date`  
> - задержек (`Timer`, `asyncAfter`, `sleep`)  
> - длительности анимаций  
> - таймаутов  
> Для красивого вывода — `DateComponentsFormatter`.  
> Для человекочитаемых констант — добавляйте расширения.
