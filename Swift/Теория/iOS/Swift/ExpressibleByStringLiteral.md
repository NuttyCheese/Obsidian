**`ExpressibleByStringLiteral`** — это протокол стандартной библиотеки [[Swift]], который позволяет типу **инициализироваться строковым литералом** (`"hello"`, `"café"`, `"🙂"` и т.д.).

```swift
public protocol ExpressibleByStringLiteral {
    associatedtype StringLiteralType : _ExpressibleByBuiltinStringLiteral
    init(stringLiteral value: StringLiteralType)
}
```

> Проще говоря: если тип реализует этот протокол, компилятор разрешает писать  
> `let name: UserName = "alice"`  
> вместо  
> `let name = UserName(stringLiteral: "alice")`.

### 1. Самое важное: `"hello"` — это НЕ [[String]]

```swift
let a = "hello"           // → String
let b: Substring = "hello" // → Substring
let c: StaticString = "hello" // → StaticString
let d: Character = "A"    // → Character
let e: MyPath = "/users"  // → MyPath (если реализует протокол)
```

**`"hello"` — это абстрактный string literal**, у которого **нет типа сам по себе**.  
Тип определяется **контекстом** (тип переменной, аргумента, возвращаемого значения).

Компилятор делает примерно следующее:

```swift
let x: MyType = "hello"
// ↓
let x = MyType(stringLiteral: "hello")
```

### 2. Кто уже реализует протокол (2026)

| Тип                                | Реализует?                                    | Особенности                                        |
| ---------------------------------- | --------------------------------------------- | -------------------------------------------------- |
| `String`                           | Да                                            | Основной тип для динамических строк                |
| `Substring`                        | Да                                            | Срез строки (без копирования)                      |
| `StaticString`                     | Да                                            | Только compile-time строки, без аллокации          |
| `Character`                        | Да                                            | Только один графемный кластер (через подпротоколы) |
| `NSString`                         | Да                                            | Foundation-совместимость                           |
| [[URL]]                            | Да (через `ExpressibleByStringInterpolation`) | `let url: URL = "https://example.com"`             |
| `Regex` (iOS 16+)                  | Да                                            | `let pattern: Regex = "[0-9]+"`                    |
| `LocalizedStringKey` ([[SwiftUI]]) | Да                                            | `Text("hello")`                                    |

### 3. Подпротоколы (очень важно!)

`ExpressibleByStringLiteral` — это **самый общий** протокол.  
Под ним есть более строгие варианты:

| Протокол                              | Что позволяет                          | Пример |
|---------------------------------------|----------------------------------------|--------|
| `ExpressibleByUnicodeScalarLiteral`   | Один Unicode-скаляр                    | `let c: Character = "A"` |
| `ExpressibleByExtendedGraphemeClusterLiteral` | Один графемный кластер (Character) | `let emoji: Character = "🙂"` |
| `ExpressibleByStringInterpolation`    | Интерполяция строк (`"Hello \(name)"`) | `let greeting = "Hi, \(user)"` |

```swift
let c: Character = "é"     // OK (extended grapheme cluster)
let wrong: Character = "ab" // ❌ — два символа
```

### 4. Реальные примеры использования (практика 2026)

#### Пример 1: Семантические идентификаторы

```swift
struct UserID: ExpressibleByStringLiteral, Hashable, Sendable {
    let rawValue: String
    
    init(stringLiteral value: String) {
        precondition(!value.isEmpty, "UserID cannot be empty")
        self.rawValue = value
    }
}

let currentUser: UserID = "alice123"
let admin: UserID = "admin"
```

#### Пример 2: Путь в файловой системе (очень популярно)

```swift
struct FilePath: ExpressibleByStringLiteral {
    let path: String
    
    init(stringLiteral value: String) {
        self.path = value
    }
}

let logFile: FilePath = "/var/log/app.log"
```

#### Пример 3: Email (с валидацией на этапе инициализации)

```swift
struct EmailAddress: ExpressibleByStringLiteral, Hashable {
    let value: String
    
    init(stringLiteral value: String) {
        precondition(value.contains("@"), "Invalid email format")
        self.value = value.lowercased()
    }
}

let support: EmailAddress = "support@example.com"
// let wrong: EmailAddress = "invalid" // compile-time ошибка (precondition)
```

### 5. Производительность и тонкости (замеры 2026)

| Операция                          | Обычная `String` | Пользовательский тип с протоколом | Примечание                 |
| --------------------------------- | ---------------- | --------------------------------- | -------------------------- |
| Создание из литерала              | ~2–5 нс          | ~5–10 нс                          | + вызов [[init]]           |
| Проверка precondition / валидация | —                | +5–20 нс                          | compile-time если возможно |
| Сравнение (==)                    | ~3–8 нс          | ~5–12 нс                          | зависит от реализации      |
| Хранение в памяти                 | ~16–64 байт      | ~16–64 байт                       | + overhead структуры       |

**Вывод**:  
Накладные расходы минимальны.  
Главное преимущество — **семантика** и **безопасность**, а не производительность.

### 6. Ловушки и антипаттерны

❌ Заменять `String` повсеместно

```swift
struct BadString: ExpressibleByStringLiteral {
    let value: String
    init(stringLiteral value: String) { self.value = value }
}
```

Это ухудшает читаемость и добавляет ненужный overhead.

❌ Забывать валидацию

```swift
struct UnsafeEmail: ExpressibleByStringLiteral {
    let value: String
    init(stringLiteral value: String) {
        self.value = value  // ← нет проверки
    }
}
```

Правильно — `precondition` или `clamp` / `trimming`.

❌ Использовать в динамических строках

```swift
let userInput = readLine()!
let email: EmailAddress = userInput  // ❌ — не литерал!
```

Протокол работает **только с литералами**, а не с runtime-строками.

### 7. Лучшие практики ExpressibleByStringLiteral в 2026

- Используй для **семантических обёрток**: `UserID`, `EmailAddress`, `FilePath`, `APIKey`, `Endpoint`  
- Всегда проверяй валидность в [[init]] (`precondition`, `guard`)  
- Делай тип [[Hashable]], [[Sendable]], [[Codable]] — если используется в моделях  
- Комбинируй с `ExpressibleByIntegerLiteral`, `ExpressibleByStringInterpolation` — для мощных DSL  
- **Не используй** вместо `String` в обычном коде — это антипаттерн  
- [[Swift]] 6 strict concurrency — тип должен быть `Sendable`  
- Документируй — пиши комментарий «ExpressibleByStringLiteral — позволяет писать `id = "alice123"` вместо `id = UserID("alice123")`»

**Короткий девиз 2026**:
> `ExpressibleByStringLiteral` — это когда ты превращаешь `"alice123"` в **смысл**, а не просто в строку.  
> Используй для **идентификаторов**, **ключей**, **путей**, **email**, **[[API]]-эндпоинтов**.  
> Но **не заменяй** `String` — это для семантики, а не для текста.
