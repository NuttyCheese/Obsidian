**`ExpressibleByNilLiteral`** — это протокол стандартной библиотеки Swift, который позволяет типу **инициализироваться литералом `nil`**.

```swift
public protocol ExpressibleByNilLiteral {
    init(nilLiteral: ())
}
```

> Проще говоря: если тип реализует этот протокол, компилятор разрешает писать  
> `let value: MyType = nil`  
> вместо  
> `let value = MyType(nilLiteral: ())`.

### 1. Кто уже реализует протокол (2026)

**Официально и единственно важно** — `Optional`:

```swift
extension Optional: ExpressibleByNilLiteral {
    public init(nilLiteral: ()) {
        self = .none
    }
}
```

Это значит, что любой `Optional<T>` (т.е. `T?`) может быть инициализирован через `nil`:

```swift
let a: Int?    = nil
let b: String? = nil
let c: User?   = nil
```

**nil — это не значение**.  
Это **литерал языка**, который компилятор превращает в `.none` на этапе компиляции, если тип соответствует протоколу.

### 2. Как компилятор обрабатывает `nil`

```swift
let x: Int? = nil
// ↓ компилятор делает примерно это:
let x = Optional<Int>(nilLiteral: ())
     = Optional<Int>.none
```

А если тип **не** реализует протокол?

```swift
let y: Int = nil     // Ошибка компиляции
let z: String = nil  // Ошибка компиляции
```

**Ключевой вывод**:  
`nil` — это **не константа** и **не Optional.none**.  
Это **специальный литерал**, который имеет смысл **только в контексте** типа, реализующего `ExpressibleByNilLiteral`.

### 3. Реальные примеры использования (практика 2026)

#### Пример 1: Семантический «может быть пустым» (без Optional)

```swift
struct UserID: ExpressibleByNilLiteral, Hashable, Sendable {
    let value: String?
    
    init(_ value: String?) {
        self.value = value
    }
    
    init(nilLiteral: ()) {
        self.value = nil
    }
}

let guest: UserID = nil           // → UserID(value: nil)
let admin: UserID = "admin123"    // → UserID(value: "admin123")
```

Читается очень естественно, но при этом **не** является `Optional`.

#### Пример 2: Флаг «отключено / не задано»

```swift
struct DarkMode: ExpressibleByNilLiteral {
    let enabled: Bool?
    
    init(booleanLiteral value: Bool) {
        self.enabled = value
    }
    
    init(nilLiteral: ()) {
        self.enabled = nil  // "не задано пользователем"
    }
}

let systemDefault: DarkMode = nil
let forcedOn: DarkMode = true
```

#### Пример 3: «Нет значения» в конфигурации

```swift
struct CacheTTL: ExpressibleByNilLiteral {
    let seconds: TimeInterval?
    
    init(integerLiteral value: Int) {
        self.seconds = TimeInterval(value)
    }
    
    init(nilLiteral: ()) {
        self.seconds = nil  // "без кэша / вечный"
    }
}

let noCache: CacheTTL = nil
let fiveMinutes: CacheTTL = 300
```

### 4. Почему это мощно (и почему редко используется)

**Плюсы**:
- Очень читаемый и декларативный синтаксис
- Позволяет создавать **семантические обёртки** над Optional
- Идеально для DSL, конфигураций, Result Builders
- Компилятор проверяет всё на этапе сборки

**Минусы / ограничения**:
- Тип **не становится Optional** — нет `if let`, `??`, `map`, `flatMap`
- Нет автоматического unwrap
- Легко запутаться, если переусердствовать
- Не заменяет `Optional` — это **дополнение**, а не замена

### 5. Сравнение: Optional vs ExpressibleByNilLiteral

| Критерий                        | Optional<T>                          | Пользовательский тип с протоколом     |
|---------------------------------|--------------------------------------|----------------------------------------|
| Может быть `nil`                | Да                                   | Да (если сам реализует)                |
| Синтаксис присваивания `nil`    | `let x: T? = nil`                    | `let x: MyType = nil`                  |
| Доступен `if let`, `guard let`  | Да                                   | Нет                                    |
| Поддерживает `??`, `map`, `flatMap` | Да                               | Нет                                    |
| Является Optional               | Да                                   | Нет                                    |
| Лучше всего подходит для        | Передача «значение или отсутствие»   | Семантические флаги / состояния        |

### 6. Лучшие практики ExpressibleByNilLiteral в 2026

- **Используй**, если хочешь **красивый и декларативный** синтаксис для «может быть не задано»
- **Назови тип осмысленно**: `OptionalUserID`, `NullableTTL`, `UnsettableFlag`
- **Комбинируй** с другими literal-протоколами (`ExpressibleByIntegerLiteral`, `ExpressibleByStringLiteral`) — очень мощно в DSL
- **Не заменяй** `Optional` повсеместно — это антипаттерн
- **Swift 6 strict concurrency** — тип должен быть `Sendable`
- **Документируйте** — пиши комментарий «ExpressibleByNilLiteral — позволяет писать `id = nil` вместо `id = UserID(nilLiteral: ())`»

**Короткий девиз 2026**:
> `ExpressibleByNilLiteral` — это когда ты превращаешь `nil` в **смысл**, а не просто в отсутствие значения.  
> Используй для **декларативных конфигураций**, **флагов**, **опциональных идентификаторов**.  
> Но **не трогай** обычный `Optional` — он для реальной «есть/нет» логики.

Удачи с красивым и выразительным кодом! 🚫