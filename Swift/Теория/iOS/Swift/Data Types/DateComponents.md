**`DateComponents`** — это структура в [[Swift]], которая представляет **разложенную дату и время по компонентам** (год, месяц, день, час, минута, секунда и т.д.).  
Она используется как **промежуточный формат** для создания дат, вычислений интервалов и извлечения частей из `Date`.

В 2026 году `DateComponents` остаётся **основным инструментом** для всех операций с календарями, планировщиками, напоминаниями, фильтрами по датам и любыми вычислениями вида «через 3 месяца», «разница в днях» и т.д.

### 1. Ключевые факты о DateComponents (актуально 2026)

| Свойство / Поведение                | Значение / Особенность                                     | Важные детали                                           |
| ----------------------------------- | ---------------------------------------------------------- | ------------------------------------------------------- |
| Тип                                 | [[struct]] ([[value type]], копируется)                    | Полностью потокобезопасна                               |
| Основное назначение                 | «Разложить» `Date` на части или «собрать» `Date` из частей | Не хранит часовой пояс и календарь — это в [[Calendar]] |
| Все компоненты — опциональные       | [[Int]]`?` ([[nil]] = не задан)                            | `components.day = 27` — только день задан               |
| Эпоха не важна                      | Не привязана к конкретному времени — только компоненты     | Можно задать `year = 9999` — это валидно                |
| Совместимость с разными календарями | Да — зависит от `Calendar` при создании/чтении             | Григорианский, исламский, еврейский и т.д.              |

### 2. Самые частые способы создания DateComponents

```swift
// Вариант 1 — пустой + ручное заполнение (самый популярный)
var components = DateComponents()
components.year   = 2026
components.month  = 2
components.day    = 18
components.hour   = 14
components.minute = 30

// Вариант 2 — через именованные параметры (очень читаемо)
let birthday = DateComponents(
    calendar: .current,
    year: 1995,
    month: 7,
    day: 15,
    hour: 8,
    minute: 0
)

// Вариант 3 — извлечение из существующей даты (самый частый в реальном коде)
let now = Date()
let todayComponents = Calendar.current.dateComponents(
    [.year, .month, .day, .hour, .minute, .second],
    from: now
)
```

### 3. Самые полезные операции с DateComponents (2026 топ)

#### 3.1. Создание даты из компонентов

```swift
if let specificDate = Calendar.current.date(from: components) {
    print("Дата:", specificDate)
}
```

#### 3.2. Добавление/вычитание интервалов (самый частый сценарий)

```swift
let now = Date()

// Через 3 месяца и 2 дня
var add = DateComponents()
add.month = 3
add.day   = 2
let future = Calendar.current.date(byAdding: add, to: now)

// 2 недели назад
var subtract = DateComponents()
subtract.weekOfYear = -2
let past = Calendar.current.date(byAdding: subtract, to: now)
```

#### 3.3. Разница между датами (самый популярный вопрос)

```swift
let start = Date()
let end = Calendar.current.date(byAdding: .day, value: 45, to: start)!

let diff = Calendar.current.dateComponents(
    [.year, .month, .day, .hour, .minute],
    from: start,
    to: end
)

print("Разница:")
print("Годы:   \(diff.year ?? 0)")
print("Месяцы: \(diff.month ?? 0)")
print("Дни:    \(diff.day ?? 0)")
print("Часы:   \(diff.hour ?? 0)")
print("Минуты: \(diff.minute ?? 0)")
```

**Важно**:  
`.day` возвращает **полные дни** (игнорируя время).  
Если нужны точные секунды — используй `end.timeIntervalSince(start)`.

#### 3.4. Получение только даты (без времени) — популярный трюк

```swift
let today = Calendar.current.dateComponents([.year, .month, .day], from: Date())
let startOfDay = Calendar.current.date(from: today)!
```

### 4. Полный реальный пример 2026 года (ViewModel + [[async]])

```swift
@MainActor
class EventViewModel: ObservableObject {
    
    @Published var selectedDate = Date()
    @Published var events: [Event] = []
    @Published var isLoading = false
    
    func loadEvents() async {
        isLoading = true
        
        do {
            // Симуляция загрузки
            try await Task.sleep(nanoseconds: 1_000_000_000) // 1 сек
            
            let calendar = Calendar.current
            let components = calendar.dateComponents([.year, .month, .day], from: selectedDate)
            
            // Загружаем события только за выбранный день
            let startOfDay = calendar.date(from: components)!
            let endOfDay = calendar.date(byAdding: .day, value: 1, to: startOfDay)!
            
            // Здесь обычно запрос к API / Core Data
            events = fetchEvents(from: startOfDay, to: endOfDay)
            
        } catch {
            // обработка ошибки
        }
        
        isLoading = false
    }
}
```

### 5. Лучшие практики DateComponents в Swift 2026

- **Всегда** используй `Calendar.current` — учитывает локаль и пояс пользователя  
- **Не создавай DateComponents вручную**, если можно использовать `Calendar.date(byAdding:...)`  
- **Для хранения дат** — храни `Date`, а не `DateComponents` (меньше ошибок)  
- **Для фильтров/запросов** — используй `dateComponents([.year, .month, .day], from: date)` — это начало дня  
- **Для сравнения дат без времени** — сравнивай `startOfDay`  
- **Swift 6 strict concurrency** — `DateComponents` полностью `Sendable` и безопасен  
- **Документируйте** — пиши комментарий «DateComponents — компоненты выбранной даты без времени»

**Короткий девиз 2026**:
> `DateComponents` — это **кирпичики даты**: год, месяц, день, час...  
> В 2026 году используй его для:  
> - создания дат из частей  
> - вычисления интервалов (через `date(byAdding:)`)  
> - получения «начала дня»  
> - фильтров и запросов по датам  
> `Date` — это момент, `DateComponents` — его разложение, `Calendar` — инструмент для работы с ними.
