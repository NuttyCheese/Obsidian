- **`TimeInterval`** — это **тип-синоним** для [[Double]].
    
- Представляет **количество секунд**, прошедших между двумя событиями или датами.
    
- Используется для:
    
    - измерения длительности операций,
        
    - вычислений с `Date`,
        
    - таймеров ([[Timer]]),
        
    - анимаций.
        

```swift
typealias TimeInterval = Double
```

---

## 2. Создание TimeInterval

```swift
let interval: TimeInterval = 3600 // 1 час в секундах
let fiveMinutes: TimeInterval = 5 * 60 // 5 минут
let oneSecond: TimeInterval = 1
```

---

## 3. Использование с Date

### Разница между датами

```swift
let startDate = Date()
// какой-то код
let endDate = Date()

let duration: TimeInterval = endDate.timeIntervalSince(startDate)
print("Прошло секунд: \(duration)")
```

### Добавление/вычитание времени

```swift
let now = Date()
let oneHourLater = now.addingTimeInterval(3600) // через 1 час
let tenMinutesEarlier = now.addingTimeInterval(-600) // 10 минут назад
```

---

## 4. Использование с Timer

```swift
Timer.scheduledTimer(withTimeInterval: 2.0, repeats: true) { timer in
    print("Таймер сработал каждые 2 секунды")
}
```

---

## 5. Конвертация TimeInterval в часы/минуты/секунды

```swift
let duration: TimeInterval = 3665 // 1 час 1 минута 5 секунд

let hours = Int(duration) / 3600
let minutes = (Int(duration) % 3600) / 60
let seconds = Int(duration) % 60

print("\(hours)ч \(minutes)м \(seconds)с") // 1ч 1м 5с
```

---

## 6. Особенности

- **TimeInterval всегда в секундах**, дробная часть — доли секунды.
    
- Используется для операций с `Date`, таймерами и анимациями.
    
- Может быть отрицательным для прошлого времени.
    

---

## 7. Итог

- `TimeInterval` = `Double`, секунды между событиями.
    
- Применяется для **измерения длительности**, **добавления/вычитания времени**, таймеров.
    
- Совместно с `Date` позволяет легко вычислять интервалы.
    

---
