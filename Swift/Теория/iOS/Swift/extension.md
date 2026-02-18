Вот несколько полезных и часто используемых **расширений (extensions)** для типа `Date` в Swift-приложениях 2025–2026 годов.  
Они решают типичные задачи: начало/конец дня/месяца/года, удобное форматирование, сравнение по датам без времени, добавление интервалов и т.д.

```swift
extension Date {
    
    // MARK: - Компоненты (удобные вычисляемые свойства)
    
    var year: Int {
        Calendar.current.component(.year, from: self)
    }
    
    var month: Int {
        Calendar.current.component(.month, from: self)
    }
    
    var day: Int {
        Calendar.current.component(.day, from: self)
    }
    
    var hour: Int {
        Calendar.current.component(.hour, from: self)
    }
    
    var minute: Int {
        Calendar.current.component(.minute, from: self)
    }
    
    var weekday: Int {
        Calendar.current.component(.weekday, from: self)
    }
    
    // MARK: - Начало и конец периода
    
    /// Начало текущего дня (00:00:00)
    var startOfDay: Date {
        Calendar.current.startOfDay(for: self)
    }
    
    /// Конец текущего дня (23:59:59)
    var endOfDay: Date {
        Calendar.current.date(byAdding: .day, value: 1, to: startOfDay)!
            .addingTimeInterval(-1)
    }
    
    /// Начало текущего месяца
    var startOfMonth: Date {
        Calendar.current.date(from: Calendar.current.dateComponents([.year, .month], from: self))!
    }
    
    /// Конец текущего месяца
    var endOfMonth: Date {
        Calendar.current.date(byAdding: .month, value: 1, to: startOfMonth)!
            .addingTimeInterval(-1)
    }
    
    /// Начало текущего года
    var startOfYear: Date {
        Calendar.current.date(from: Calendar.current.dateComponents([.year], from: self))!
    }
    
    // MARK: - Добавление интервалов (удобные методы)
    
    func adding(days: Int) -> Date {
        Calendar.current.date(byAdding: .day, value: days, to: self)!
    }
    
    func adding(months: Int) -> Date {
        Calendar.current.date(byAdding: .month, value: months, to: self)!
    }
    
    func adding(years: Int) -> Date {
        Calendar.current.date(byAdding: .year, value: years, to: self)!
    }
    
    // MARK: - Сравнение дат без учёта времени
    
    func isSameDay(as other: Date) -> Bool {
        Calendar.current.isDate(self, inSameDayAs: other)
    }
    
    func isToday() -> Bool {
        Calendar.current.isDateInToday(self)
    }
    
    func isTomorrow() -> Bool {
        Calendar.current.isDateInTomorrow(self)
    }
    
    func isYesterday() -> Bool {
        Calendar.current.isDateInYesterday(self)
    }
    
    func isWeekend() -> Bool {
        Calendar.current.isDateInWeekend(self)
    }
    
    // MARK: - Удобное форматирование (локализованное)
    
    func formatted(style: Date.FormatStyle) -> String {
        self.formatted(style)
    }
    
    func shortDate() -> String {
        self.formatted(date: .abbreviated, time: .omitted)
    }
    
    func shortTime() -> String {
        self.formatted(date: .omitted, time: .shortened)
    }
    
    func mediumDateTime() -> String {
        self.formatted(date: .abbreviated, time: .shortened)
    }
    
    // MARK: - Относительное время (в стиле "5 минут назад")
    
    func relativeString(to now: Date = .now) -> String {
        let formatter = RelativeDateTimeFormatter()
        formatter.unitsStyle = .short
        formatter.locale = .current
        return formatter.localizedString(for: self, relativeTo: now)
    }
    
    // MARK: - Время суток (утро, день, вечер, ночь)
    
    var timeOfDay: String {
        let hour = Calendar.current.component(.hour, from: self)
        switch hour {
        case 5..<12:   return "утро"
        case 12..<18:  return "день"
        case 18..<23:  return "вечер"
        default:       return "ночь"
        }
    }
}
```

### Примеры использования (2026 стиль)

```swift
let now = Date()

print(now.shortDate())          // "18 февр. 2026"
print(now.mediumDateTime())     // "18 февр. 2026, 14:35"
print(now.relativeString())     // "только что" или "5 минут назад"

let birthday = DateComponents(calendar: .current, year: 1995, month: 7, day: 15).date!
print(birthday.isSameDay(as: DateComponents(calendar: .current, month: 7, day: 15).date!)) // true

let nextWeek = now.adding(days: 7)
print(nextWeek.relativeString()) // "через неделю"

let meeting = now.adding(hours: 2)
print(meeting.timeOfDay) // "день" или "вечер"
```

### Краткий итог — самые полезные методы

| Метод / свойство       | Что возвращает                          | Когда использовать |
|-------------------------|-----------------------------------------|---------------------|
| `startOfDay`            | 00:00:00 текущего дня                   | Фильтры по дням     |
| `endOfDay`              | 23:59:59 текущего дня                   | Диапазоны запросов  |
| `startOfMonth`          | 1-е число месяца 00:00                  | Отчёты по месяцам   |
| `adding(days:)`         | Дата + N дней                           | Сдвиг периода       |
| `isSameDay(as:)`        | true, если одна и та же дата            | Сравнение без времени |
| `isToday` / `isTomorrow`| Быстрые проверки                        | UI ("сегодня", "завтра") |
| `relativeString()`      | "5 минут назад", "через 2 дня"          | Чаты, уведомления    |
| `formatted(...)`        | Локализованная строка                   | Отображение в UI    |

Эти расширения покрывают ~90 % реальных задач с датами в iOS-приложениях в 2025–2026 годах.

Если хочешь — могу добавить ещё более специфические расширения:
- работа с кварталами / финансовыми периодами
- разница в рабочих днях
- проверка високосного года
- форматирование "сегодня в 14:30", "вчера", "5 февраля"

Просто скажи, что именно нужно!