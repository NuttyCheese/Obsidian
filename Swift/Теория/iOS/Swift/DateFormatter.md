**`DateFormatter`** — это мощный и часто используемый класс в Foundation, который отвечает за **форматирование** объекта `Date` в строку (для показа пользователю) и **парсинг** строки обратно в `Date`.

В 2026 году это **единственный рекомендуемый** способ отображать даты/время в UIKit-приложениях и обрабатывать ввод дат от пользователя.

### 1. Почему DateFormatter — must-have

| Задача / Сценарий                             | Почему без DateFormatter сложно                          | Как DateFormatter решает в 2026 |
|-----------------------------------------------|----------------------------------------------------------|---------------------------------|
| Показать дату в UI (профиль, чат, событие)    | `Date` — это просто timestamp, нечитаемый для человека   | `formatter.string(from: date)`  |
| Парсинг даты из API / текстового поля         | JSON / пользователь вводит строку "27.08.2025"           | `formatter.date(from: string)`  |
| Локализация (ru, en, fr, ja и т.д.)           | Разные форматы, порядок день/месяц/год, названия месяцев | `formatter.locale = Locale.current` |
| Учёт часового пояса                           | Пользователь в Москве, сервер в UTC                      | `formatter.timeZone = .current` или `.gmt` |
| Предопределённые стили (short, medium, long)  | Не нужно писать формат вручную                          | `dateStyle = .medium`, `timeStyle = .short` |

### 2. Самые популярные стили и шаблоны (2026 стандарт)

#### Предопределённые стили (самый читаемый и локализованный способ)

```swift
let formatter = DateFormatter()
formatter.dateStyle = .medium    // 27 авг. 2025
formatter.timeStyle = .short     // 14:30
let now = Date()
print(formatter.string(from: now)) // "27 авг. 2025, 14:30" (в русской локали)
```

Варианты `dateStyle` / `timeStyle`:

| Стиль     | Пример (27 августа 2025, 14:30, ru_RU)      | Когда использовать |
|-----------|---------------------------------------------|---------------------|
| `.none`   | (только время или только дата)              | Когда нужно показать только одну часть |
| `.short`  | 27.08.25 / 14:30                            | Компактный ввод/вывод |
| `.medium` | 27 авг. 2025 / 14:30                        | Самый частый в приложениях |
| `.long`   | 27 августа 2025 / 14:30:00 GMT+3            | Подробный текст |
| `.full`   | среда, 27 августа 2025 г. / 14:30:00 GMT+3  | Формальные документы |

#### Кастомный формат через `dateFormat` (когда нужен точный контроль)

```swift
let formatter = DateFormatter()
formatter.locale = Locale(identifier: "ru_RU")
formatter.dateFormat = "dd MMMM yyyy 'в' HH:mm"  // 27 августа 2025 в 14:30
print(formatter.string(from: Date()))
```

Популярные шаблоны:

| Шаблон              | Пример (27 августа 2025, 14:30) | Описание |
|---------------------|----------------------------------|----------|
| `dd.MM.yyyy`        | 27.08.2025                      | День.Месяц.Год |
| `dd MMMM yyyy`      | 27 августа 2025                 | Полный месяц |
| `d MMM yy`          | 27 авг. 25                      | Короткий стиль |
| `HH:mm:ss`          | 14:30:45                        | Только время |
| `yyyy-MM-dd HH:mm`  | 2025-08-27 14:30                | ISO-подобный, для API |
| `E, d MMM yyyy`     | ср, 27 авг. 2025                | День недели + дата |

### 3. Полный реальный пример 2026 года (ViewModel + async + DateFormatter)

```swift
@MainActor
class EventViewModel: ObservableObject {
    
    @Published var events: [Event] = []
    @Published var formattedDate: String = ""
    
    private let dateFormatter: DateFormatter = {
        let formatter = DateFormatter()
        formatter.locale = Locale(identifier: "ru_RU")
        formatter.dateStyle = .medium
        formatter.timeStyle = .short
        return formatter
    }()
    
    func loadEvents(for date: Date) async {
        do {
            let events = try await fetchEvents(for: date)
            
            await MainActor.run {
                self.events = events
                self.formattedDate = dateFormatter.string(from: date)
            }
        } catch {
            await MainActor.run {
                // показать ошибку
            }
        }
    }
}
```

### 4. Парсинг строк в Date (самый частый вопрос)

```swift
let formatter = DateFormatter()
formatter.locale = Locale(identifier: "ru_RU")
formatter.dateFormat = "dd.MM.yyyy HH:mm"

// Парсинг
let input = "27.08.2025 14:30"
if let date = formatter.date(from: input) {
    print("Дата:", date)
} else {
    print("Неверный формат")
}
```

**Ловушка**:  
Если локаль или формат не совпадают — `date(from:)` вернёт `nil`.  
Всегда задавай `locale` и `dateFormat` явно.

### 5. Лучшие практики DateFormatter в Swift 2026

- **Создавай DateFormatter один раз** — это тяжёлый объект, храни как свойство или static  
- **Всегда указывай `locale`** — иначе формат будет зависеть от устройства пользователя  
- **Для API и хранения** — используй ISO8601DateFormatter (стандартный, без локали)  
- **Для UI** — используй предопределённые стили (`.medium`, `.short`) — они автоматически локализуются  
- **Не храни Date в виде строки** — храни как `Date`, форматируй только при отображении  
- **Для относительных дат** — используй `RelativeDateTimeFormatter` (iOS 13+)  
- **Swift 6 strict concurrency** — `DateFormatter` полностью безопасен (`Sendable`)  
- **Документируйте** — пиши комментарий «DateFormatter — форматирование даты в русской локали»

**Короткий девиз 2026**:
> `DateFormatter` — это **мост** между `Date` (момент времени) и строкой, понятной человеку.  
> В 2026 году:  
> - создавай один раз и переиспользуй  
> - используй `.medium`/`.short` для UI  
> - задавай `locale` и `timeZone` явно  
> - для API — ISO8601DateFormatter  
> Это **единственный правильный** способ показывать и парсить даты в приложении.

Удачи с красивыми и локализованными датами в твоём UI! 🗓️