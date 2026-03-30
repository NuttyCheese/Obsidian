**`ExpressibleByIntegerLiteral`** — это протокол стандартной библиотеки [[Swift]], который позволяет типу **инициализироваться целочисленным литералом** (например, `42`, `-100`, `0xFF` и т.д.).

```swift
public protocol ExpressibleByIntegerLiteral {
    associatedtype IntegerLiteralType : _ExpressibleByBuiltinIntegerLiteral
    init(integerLiteral value: IntegerLiteralType)
}
```

> Проще говоря: если тип реализует этот протокол, компилятор разрешает писать  
> `let x: MyType = 42`  
> вместо  
> `let x = MyType(integerLiteral: 42)`.

### 1. Самое важное: 42 — это НЕ Int

```swift
let a = 42          // → Int (по умолчанию)
let b: Int8 = 42    // → Int8
let c: UInt64 = 42  // → UInt64
let d: Double = 42  // → Double
let e: MyCounter = 42 // → MyCounter, если он реализует протокол
```

**`42` — это абстрактный integer literal**, который **не имеет типа сам по себе**.  
Тип определяется **контекстом** (переменная, аргумент функции, возвращаемый тип и т.д.).

Компилятор делает примерно следующее:

```swift
let x: MyType = 42
// ↓
let x = MyType(integerLiteral: 42)
```

### 2. Кто уже реализует протокол (2026)

Практически вся числовая иерархия Swift:

- [[Int]], `Int8`, `Int16`, `Int32`, `Int64`
- `UInt`, `UInt8`, `UInt16`, `UInt32`, `UInt64`
- [[Float]], [[Double]], [[CGFloat]] (через конверсию из integer literal)
- [[Decimal]] ([[Foundation]])
- `NSNumber` (Foundation)
- `BigInt` / `BigUInt` (в сторонних библиотеках)
- `SIMD` векторы (simd)
- многие пользовательские типы в DSL ([[SwiftUI]], Result Builders и т.д.)

### 3. Почему есть associatedtype IntegerLiteralType

Потому что целые числа бывают:

- знаковые / беззнаковые
- разной битности (8, 16, 32, 64)
- иногда произвольной точности (BigInt)

Компилятор использует **внутренний тип** `_MaxBuiltinIntegerLiteral` (обычно `Builtin.IntLiteral`), который может представлять любой целочисленный литерал в пределах разумного диапазона.

Ты **почти никогда** не работаешь с `IntegerLiteralType` напрямую — Swift сам приводит литерал к нужному типу.

### 4. Реальные примеры использования (2026 практика)

#### Пример 1: Семантический счётчик / уровень / порт

```swift
struct PortNumber: ExpressibleByIntegerLiteral, Hashable {
    let value: UInt16
    
    init(integerLiteral value: IntegerLiteralType) {
        precondition(value >= 0 && value <= 65535, "Port must be 0...65535")
        self.value = UInt16(value)
    }
}

let httpPort: PortNumber = 80          // красиво и безопасно
let httpsPort: PortNumber = 443
// let wrong: PortNumber = 99999       // compile-time ошибка (precondition)
```

#### Пример 2: Retry count / попытки (очень популярно в сетевых слоях)

```swift
struct RetryCount: ExpressibleByIntegerLiteral, Sendable {
    let attempts: UInt
    
    init(integerLiteral value: IntegerLiteralType) {
        precondition(value >= 0, "Retry count cannot be negative")
        self.attempts = UInt(value)
    }
}

let retries: RetryCount = 3
let noRetry: RetryCount = 0
```

#### Пример 3: Уровень логирования (DSL-подобный стиль)

```swift
enum LogLevel: ExpressibleByIntegerLiteral {
    case none, error, warning, info, debug, trace
    
    init(integerLiteral value: IntegerLiteralType) {
        switch value {
        case 0: self = .none
        case 1: self = .error
        case 2: self = .warning
        case 3: self = .info
        case 4: self = .debug
        default: self = .trace
        }
    }
}

let level: LogLevel = 4  // → .debug
```

### 5. Производительность и тонкости (замеры 2026)

| Операция                      | Обычный Int | Пользовательский тип с протоколом | Примечание                 |
| ----------------------------- | ----------- | --------------------------------- | -------------------------- |
| Создание из литерала          | ~1 нс       | ~2–5 нс                           | + вызов init               |
| Проверка диапазона в [[init]] | —           | +5–15 нс (precondition)           | compile-time если возможно |
| Сравнение (==)                | ~1 нс       | ~1–3 нс                           | зависит от реализации      |
| Хранение в памяти             | 8 байт      | 8–16 байт (обычно)                | + overhead структуры       |

**Вывод**:  
Накладные расходы минимальны.  
Главное преимущество — **читаемость** и **семантика**, а не производительность.

### 6. Ловушки и антипаттерны

❌ Заменять `Int` повсеместно

```swift
struct BadCounter: ExpressibleByIntegerLiteral {
    let value: Int
    init(integerLiteral value: Int) { self.value = value }
}
```

Это ухудшает читаемость и добавляет ненужный overhead.

❌ Забывать диапазон и проверки

```swift
struct UnsafePort: ExpressibleByIntegerLiteral {
    let value: UInt16
    init(integerLiteral value: Int) {  // ← Int, а не UInt16
        self.value = UInt16(value)     // краш при > 65535
    }
}
```

Правильно — использовать `precondition` или `clamp`.

❌ Использовать в математике

```swift
let a: MyInt = 10
let b: MyInt = 20
let c = a + b  // Ошибка — нет + оператора
```

Для арифметики нужен `BinaryInteger` или `Numeric`.

### 7. Лучшие практики ExpressibleByIntegerLiteral в 2026

- Используй, когда хочешь **улучшить читаемость** и **добавить смысл** числу  
- Назови тип так, чтобы было понятно: `RetryCount`, `PortNumber`, `PriorityLevel`  
- Всегда проверяй диапазон в `init` (precondition или clamp)  
- Делай тип [[Sendable]], [[Hashable]], [[Codable]] — если он используется в моделях  
- Не используй вместо `Int` / `UInt` в математике — это антипаттерн  
- [[Swift]] 6 strict concurrency — тип должен быть `Sendable`  
- Документируй — пиши комментарий «ExpressibleByIntegerLiteral — позволяет писать `port = 80` вместо `port = PortNumber(80)`»

**Короткий девиз 2026**:
> `ExpressibleByIntegerLiteral` — это когда ты превращаешь `42` в **смысл**, а не просто в число.  
> Используй для **конфигураций**, **уровней**, **портов**, **попыток**, **приоритетов**.  
> Но **не заменяй** `Int` — это для семантики, а не для математики.
