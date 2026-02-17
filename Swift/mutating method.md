**`mutating`** — это модификатор метода, который позволяет **изменять свойства** структуры ([[struct]]) или перечисления ([[enum]]) **изнутри самого метода**.

Без `mutating` компилятор запрещает менять свойства [[Value Type]] внутри метода — даже если переменная объявлена как [[var]].

### 1. Почему mutating нужен только для value types

| Тип данных | Семантика                   | Изменение внутри метода                | Нужен ли mutating?          | Причина                         |
| ---------- | --------------------------- | -------------------------------------- | --------------------------- | ------------------------------- |
| [[struct]] | Value type (копируется)     | Да (если метод изменяет свойства)      | Да                          | Изменение копии, а не оригинала |
| [[enum]]   | [[Value Type]]              | Да (смена кейса или associated values) | Да                          | То же самое                     |
| [class]    | [[Reference Type]] (ссылка) | Да (всегда)                            | Нет                         | Изменяется объект по ссылке     |
| [[actor]]  | Reference + изоляция        | Да (внутри actor)                      | Нет (но есть `nonisolated`) | Изоляция потоков                |

**Главное правило**:
> Mutating нужен **только** для методов, которые изменяют свойства структуры или enum.  
> Для классов и actor’ов он **запрещён** и **не нужен**.

### 2. Синтаксис и базовые примеры

#### Пример 1 — Простейший mutating метод

```swift
struct Counter {
    var value: Int = 0
    
    mutating func increment() {
        value += 1
    }
    
    mutating func reset() {
        value = 0
    }
}

var counter = Counter()
counter.increment()  // OK
counter.increment()  // value = 2
counter.reset()      // value = 0

let constantCounter = Counter()
// constantCounter.increment() // Ошибка компиляции: Cannot use mutating member on immutable value
```

#### Пример 2 — Mutating метод с параметрами

```swift
struct Point {
    var x: Double
    var y: Double
    
    mutating func moveBy(dx: Double, dy: Double) {
        x += dx
        y += dy
    }
    
    mutating func scale(factor: Double) {
        x *= factor
        y *= factor
    }
}

var p = Point(x: 3, y: 4)
p.moveBy(dx: 1, dy: 2)   // → (4, 6)
p.scale(factor: 2)       // → (8, 12)
```

#### Пример 3 — Переопределение [[self]] (очень мощная фича)

```swift
struct Point {
    var x: Int
    var y: Int
    
    mutating func resetToOrigin() {
        self = Point(x: 0, y: 0)  // полная замена себя
    }
    
    mutating func mirrorX() {
        self.x = -self.x          // можно менять свойства
    }
}

var origin = Point(x: 10, y: 20)
origin.resetToOrigin()  // → (0, 0)
origin.mirrorX()        // → (0, 0) всё равно
```

#### Пример 4 — Mutating в enum (смена кейса)

```swift
enum TrafficLight {
    case red, yellow, green
    
    mutating func next() {
        switch self {
        case .red:    self = .green
        case .yellow: self = .red
        case .green:  self = .yellow
        }
    }
    
    mutating func emergencyStop() {
        self = .red
    }
}

var light = TrafficLight.red
light.next()          // → green
light.next()          // → yellow
light.emergencyStop() // → red
```

#### Пример 5 — Mutating метод в [[generic]] структуре

```swift
struct Stack<Element> {
    private var items: [Element] = []
    
    mutating func push(_ item: Element) {
        items.append(item)
    }
    
    mutating func pop() -> Element? {
        items.popLast()
    }
    
    mutating func clear() {
        items.removeAll()
    }
}

var intStack = Stack<Int>()
intStack.push(1)
intStack.push(2)
intStack.pop()     // → 2
intStack.clear()   // пусто
```

### 3. Mutating и протоколы

Протокол может требовать mutating метод — тогда conforming тип должен его реализовать как mutating.

```swift
protocol Resettable {
    mutating func reset()
}

struct Counter: Resettable {
    var count = 0
    
    mutating func reset() {
        count = 0
    }
}

class SecureCounter: Resettable {  // Ошибка!
    var count = 0
    
    func reset() {  // ❌ метод не mutating
        count = 0
    }
}
```

**Правило**:  
Если протокол требует `mutating func`, то классы **не могут** его реализовать (потому что классы не нуждаются в mutating).

Решение: использовать [[AnyObject]] + non-mutating

```swift
protocol Resettable: AnyObject {
    func reset()
}
```

### 4. Реальные сценарии в iOS-разработке (2026)

#### Сценарий 1 — Управление состоянием экрана (ViewModel)

```swift
@MainActor
class ProfileViewModel: ObservableObject {
    @Published var user: User?
    @Published var isLoading = false
    
    private var profileService: ProfileService
    
    init(profileService: ProfileService) {
        self.profileService = profileService
    }
    
    func loadProfile() async {
        isLoading = true
        do {
            user = try await profileService.fetchProfile()
        } catch {
            // обработка ошибки
        }
        isLoading = false
    }
}
```

Здесь `mutating` не нужен — `@Published` изменяется через reference semantics.

#### Сценарий 2 — Mutable состояние в структуре ([[SwiftUI]])

```swift
struct SettingsView: View {
    @State private var settings = Settings()
    
    var body: some View {
        Form {
            Toggle("Тёмная тема", isOn: $settings.isDarkMode)
            Stepper("Размер текста: \(settings.fontSize)", value: $settings.fontSize, in: 12...24)
        }
    }
}

struct Settings {
    var isDarkMode = false
    var fontSize: Int = 16
    
    mutating func toggleDarkMode() {
        isDarkMode.toggle()
    }
    
    mutating func increaseFontSize() {
        fontSize += 1
    }
}
```

`@State` делает структуру mutable → можно вызывать mutating методы.

### 5. Таблица: mutating vs non-mutating

| Ситуация                              | Mutating нужен? | Можно ли вызвать на let? | Пример |
|---------------------------------------|------------------|---------------------------|--------|
| Изменение свойства структуры          | Да               | Нет                       | `mutating func increment()` |
| Изменение свойства класса             | Нет              | Да                        | `func updateName(_ name: String)` |
| Смена кейса enum                      | Да               | Нет                       | `mutating func next()` |
| Только чтение свойств                 | Нет              | Да                        | `func description() -> String` |
| Метод, который возвращает новый объект| Нет              | Да                        | `func movedBy(dx: Int) -> Point` |

### 6. Типичные ошибки и ловушки

| Ошибка                                      | Последствия                                  | Как избежать |
|---------------------------------------------|----------------------------------------------|--------------|
| Вызов mutating на let                       | Ошибка компиляции                            | Использовать `var` для изменяемых экземпляров |
| Mutating в классе                           | Ошибка компиляции                            | Убрать `mutating` |
| Mutating в протоколе без AnyObject          | Классы не могут реализовать                 | Добавить `: AnyObject` или убрать mutating |
| Забыли mutating при изменении свойства      | Ошибка компиляции                            | Добавить mutating |
| Mutating в computed property                | Ошибка компиляции                            | Computed properties не могут быть mutating |

### 7. Лучшие практики 2026 года

- Используй `mutating` **только** когда метод **изменяет** свойства структуры/enum
- Для immutable поведения — возвращай новый экземпляр (функциональный стиль):

```swift
struct Point {
    var x: Int
    var y: Int
    
    func movedBy(dx: Int, dy: Int) -> Point {
        Point(x: x + dx, y: y + dy)
    }
}
```

- В SwiftUI — `@State`, `@Binding`, `@ObservedObject` делают mutating невидимым
- В протоколах — если метод mutating, то классы его не реализуют → добавляй `: AnyObject`
- Для производительности — избегай лишних копирований (маленькие структуры — хорошо)
- В actor’ах — mutating не используется, вместо этого `nonisolated` или `isolated`

**Короткий девиз**:
> «Mutating — это способ сказать структуре или enum: «Ты можешь измениться внутри метода».  
> Без него value type остаётся immutable внутри метода — как и положено копируемому типу.»
