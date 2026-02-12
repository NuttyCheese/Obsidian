## 1. Что такое `Calendar`

**`Calendar`** — это структура, которая позволяет:

- работать с **компонентами даты** (год, месяц, день, час, минута),
    
- выполнять **вычисления с датами** (добавление/вычитание дней, месяцев, лет),
    
- учитывать **локаль и часовой пояс**,
    
- поддерживать разные **календарные системы** (григорианский, буддийский, исламский и др.).
    

> Проще: `Calendar` = логика календаря для работы с [[Date]].

---

## 2. Получение календаря

```swift
let calendar = Calendar.current // локальный календарь пользователя
let gregorian = Calendar(identifier: .gregorian) // григорианский календарь
```

---

## 3. Получение компонентов даты

```swift
let now = Date()
let calendar = Calendar.current

let year = calendar.component(.year, from: now)
let month = calendar.component(.month, from: now)
let day = calendar.component(.day, from: now)
let hour = calendar.component(.hour, from: now)
let minute = calendar.component(.minute, from: now)

print("Сегодня: \(day).\(month).\(year) \(hour):\(minute)")
```

---

## 4. Работа с [[DateComponents]]

```swift
// Создание даты через компоненты
var components = DateComponents()
components.year = 2025
components.month = 8
components.day = 27
components.hour = 12
components.minute = 30

let date = calendar.date(from: components)
print(date!) // 2025-08-27 12:30:00 +0000
```

---

## 5. Добавление и вычитание времени

```swift
let now = Date()

// Добавляем 1 день
let tomorrow = calendar.date(byAdding: .day, value: 1, to: now)!

// Вычитаем 7 дней
let lastWeek = calendar.date(byAdding: .day, value: -7, to: now)!

print("Завтра: \(tomorrow)")
print("Неделю назад: \(lastWeek)")
```

---

## 6. Разница между датами

```swift
let startDate = Date()
let endDate = calendar.date(byAdding: .hour, value: 3, to: startDate)!

let components = calendar.dateComponents([.hour, .minute], from: startDate, to: endDate)
print("Разница: \(components.hour!) часов, \(components.minute!) минут") // 3 часов, 0 минут
```

---

## 7. Особенности Calendar

- Работает с `Date` и `DateComponents`.
    
- Учитывает **локаль и часовой пояс**.
    
- Поддерживает разные типы календарей (`.gregorian`, `.buddhist`, `.islamic`, `.hebrew`).
    
- Позволяет выполнять операции **вычисления и сравнения** с датами.
    

---

## 8. Итог

- `Calendar` = инструмент для манипуляций с датами.
    
- Позволяет получать компоненты, создавать даты, добавлять/вычитать время.
    
- Используется вместе с `Date` и `DateComponents`.
    

---
