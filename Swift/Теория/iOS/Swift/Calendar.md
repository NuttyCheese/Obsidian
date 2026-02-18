**`Calendar`** в Swift — это мощный и очень часто используемый инструмент для работы с датами и временем. Он отвечает за всю «логику календаря»: компоненты даты (год, месяц, день, час...), часовые пояса, локали, разные календарные системы и математические операции над `Date`.

В 2026 году это **единственный рекомендуемый** способ выполнять любые серьёзные манипуляции с датами в UIKit и Foundation-приложениях.

### 1. Зачем нужен Calendar (ключевые задачи)

| Задача / Сценарий                             | Почему без Calendar сложно                              | Как Calendar решает в 2026 |
|-----------------------------------------------|----------------------------------------------------------|-----------------------------|
| Получить день/месяц/год из Date               | `Date` — это просто timestamp, без компонентов           | `calendar.component(.day, from: date)` |
| Добавить/вычесть дни, недели, месяцы, годы    | Учёт високосных лет, переходов месяцев, часовых поясов   | `calendar.date(byAdding: .month, value: 3, to: date)` |
| Создать дату из компонентов (27 августа 2025) | Нужно учесть локаль, пояс, високосность                  | `calendar.date(from: components)` |
| Посчитать разницу между датами                | Правильно учитывать дни/месяцы/годы                      | `calendar.dateComponents([.day, .month], from: start, to: end)` |
| Работать с разными календарями                | Григорианский, исламский, еврейский, китайский и т.д.   | `Calendar(identifier: .islamic)` |
| Учитывать локаль и часовой пояс               | Форматы даты, начало недели, DST                         | `Calendar.current` или `calendar.locale = Locale(identifier: "ru_RU")` |

### 2. Самые важные способы получения Calendar

```swift
// Самый частый — календарь пользователя (с учётом локали и пояса)
let calendar = Calendar.current

// Явно григорианский (самый распространённый в мире)
let gregorian = Calendar(identifier: .gregorian)

// Исламский календарь (для хиджры)
let islamic = Calendar(identifier: .islamic)

// Еврейский календарь
let hebrew = Calendar(identifier: .hebrew)

// Китайский лунный календарь
let chinese = Calendar(identifier: .chinese)
```

**Рекомендация 2026**:  
99% случаев — используйте `Calendar.current`.  
Явный `identifier` нужен только для специфических задач (религиозные календари, исторические расчёты).

### 3. Получение компонентов даты — самый частый сценарий

```swift
let now = Date()
let cal = Calendar.current

let year   = cal.component(.year,   from: now)   // 2026
let month  = cal.component(.month,  from: now)   // 1–12
let day    = cal.component(.day,    from: now)   // 1–31
let hour   = cal.component(.hour,   from: now)   // 0–23
let minute = cal.component(.minute, from: now)   // 0–59
let weekday = cal.component(.weekday, from: now) // 1=воскресенье, 2=понедельник... (зависит от локали!)
```

**Все доступные компоненты** (`.era`, `.year`, `.month`, `.day`, `.hour`, `.minute`, `.second`, `.nanosecond`, `.weekday`, `.weekdayOrdinal`, `.quarter`, `.weekOfMonth`, `.weekOfYear`, `.yearForWeekOfYear`, `.isLeapMonth`)

### 4. Самый популярный паттерн: работа с DateComponents

```swift
var components = DateComponents()
components.year   = 2025
components.month  = 12
components.day    = 31
components.hour   = 23
components.minute = 59

// Создаём дату из компонентов
if let newYearEve = Calendar.current.date(from: components) {
    print("Новый год: \(newYearEve)")  // 2025-12-31 23:59:00 +0000
}

// Изменяем существующую дату
let now = Date()
if let tomorrow = Calendar.current.date(byAdding: .day, value: 1, to: now) {
    print("Завтра: \(tomorrow)")
}
```

### 5. Разница между датами (самый частый вопрос)

```swift
let start = Date()
let end = Calendar.current.date(byAdding: .day, value: 10, to: start)!

let diff = Calendar.current.dateComponents([.day, .hour, .minute], from: start, to: end)
print("Прошло: \(diff.day ?? 0) дней, \(diff.hour ?? 0) часов")

// Только количество дней (игнорируя время)
let days = Calendar.current.dateComponents([.day], from: start, to: end).day ?? 0
```

### 6. Лучшие практики Calendar в Swift 2026

- **Всегда** используйте `Calendar.current` — он учитывает локаль, пояс и настройки пользователя  
- **Не храните `Calendar` как свойство** — создавайте заново при необходимости (`let cal = Calendar.current`)  
- **Ограничивайте компоненты** — запрашивайте только нужные: `[.day, .month]` вместо всего  
- **Для форматирования** — используйте `DateFormatter`, а не `Calendar` напрямую  
- **Для диапазонов дат** — Swift 6+ имеет `DateInterval`, но `Calendar` всё ещё основной инструмент  
- **Swift 6 strict concurrency** — `Calendar` полностью Sendable и безопасен  
- **Документируйте** — пиши комментарий «Calendar.current — локальный календарь для вычисления разницы дат»

**Короткий девиз 2026**:
> `Calendar` — это **мозг всех операций с датами** в Swift: компоненты, добавление/вычитание, разница, локали, пояса.  
> В 2026 году используйте `Calendar.current`, `DateComponents`, `date(byAdding:)`, `dateComponents(from:to:)` и забудьте ручной парсинг timestamp’ов.  
> Это **единственный правильный** способ работать с датами в UIKit и Foundation.

Удачи с точными и локализованными датами в твоём приложении! 📅