## 1. Что такое `DateComponents`

**`DateComponents`** — это структура, которая хранит **компоненты даты и времени отдельно**:

- год (`year`)
    
- месяц (`month`)
    
- день (`day`)
    
- час (`hour`)
    
- минута (`minute`)
    
- секунда (`second`)
    
- день недели (`weekday`)
    
- часовой пояс (`timeZone`)
    

> Проще: `DateComponents` = набор «кирпичиков», из которых строится [[Date]].

---

## 2. Создание DateComponents

### Простой пример

```swift
var components = DateComponents()
components.year = 2025
components.month = 8
components.day = 27
components.hour = 12
components.minute = 30

print(components)
```

---

### С инициализатором

```swift
let components = DateComponents(calendar: Calendar.current, year: 2025, month: 8, day: 27, hour: 12, minute: 30)
```

---

## 3. Преобразование в Date

Чтобы получить `Date` из `DateComponents`, используем [[Calendar]]:

```swift
let calendar = Calendar.current
if let date = calendar.date(from: components) {
    print(date) // 2025-08-27 12:30:00 +0000
}
```

---

## 4. Разбор Date на компоненты

Можно получить `DateComponents` из существующей даты:

```swift
let now = Date()
let calendar = Calendar.current

let components = calendar.dateComponents([.year, .month, .day, .hour, .minute], from: now)
print("Сегодня: \(components.day!)-\(components.month!)-\(components.year!) \(components.hour!):\(components.minute!)")
```

---

## 5. Использование для вычислений

### Добавление и вычитание

```swift
let now = Date()
let calendar = Calendar.current

var components = DateComponents()
components.day = 7 // через 7 дней

if let nextWeek = calendar.date(byAdding: components, to: now) {
    print("Через неделю: \(nextWeek)")
}
```

### Разница между датами

```swift
let startDate = Date()
let endDate = calendar.date(byAdding: .hour, value: 3, to: startDate)!

let difference = calendar.dateComponents([.hour, .minute], from: startDate, to: endDate)
print("Разница: \(difference.hour!) часов, \(difference.minute!) минут") // 3 часов, 0 минут
```

---

## 6. Особенности

- `DateComponents` не хранит конкретную дату — только **компоненты**.
    
- Для конверсии в `Date` нужен `Calendar`.
    
- Можно использовать для:
    
    - создания даты,
        
    - вычислений,
        
    - разборки даты на части,
        
    - установки повторяющихся событий.
        

---

## 7. Итог

- `DateComponents` = набор компонентов даты и времени.
    
- Создаём вручную или извлекаем из `Date`.
    
- Совместно с `Calendar` позволяет вычислять даты, добавлять/вычитать время.
    
- Используется в календарях, таймерах, планировщиках и для форматирования дат.
    

---
