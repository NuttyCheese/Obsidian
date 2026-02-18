**`ExpressibleByBoolLiteral`** — это протокол стандартной библиотеки [[Swift]], который позволяет типу **инициализироваться напрямую литералами `true` и `false`**.

```swift
public protocol ExpressibleByBoolLiteral {
    associatedtype BooleanLiteralType: _ExpressibleByBuiltinBooleanLiteral
    init(booleanLiteral value: BooleanLiteralType)
}
```

> Проще говоря: если тип реализует этот протокол, то компилятор разрешает писать  
> `let flag: MyType = true`  
> вместо  
> `let flag = MyType(booleanLiteral: true)`.

### 1. Кто уже реализует протокол

**Официально реализует только `Bool`:**

```swift
extension Bool: ExpressibleByBoolLiteral {
    public init(booleanLiteral value: Bool) {
        self = value
    }
}
```

Но самое интересное — **протокол открыт для всех типов**, и именно это даёт магию.

### 2. Зачем это нужно? Семантическая магия

`true` / `false` — это не просто `Bool`.  
Это **литерал**, который может означать **разные вещи** в зависимости от контекста.

```swift
// Bool
let isActive: Bool = true

// Семантический флаг (Feature Toggle)
struct FeatureFlag: ExpressibleByBoolLiteral {
    let enabled: Bool
    init(booleanLiteral value: Bool) {
        self.enabled = value
    }
}
let newUI: FeatureFlag = true          // читается как «включить новый UI»
let darkMode: FeatureFlag = false      // «тёмная тема выключена»

// Разрешения
struct Permission: ExpressibleByBoolLiteral {
    let granted: Bool
    init(booleanLiteral value: Bool) {
        self.granted = value
    }
}
let cameraAccess: Permission = true    // «доступ к камере разрешён»
```

**Читаемость кода взлетает**, потому что вместо

```swift
enableNewFeature(true)
```

мы пишем

```swift
enableNewFeature(true)  // всё ещё работает, но смысл скрыт
```

а с типом:

```swift
enableNewFeature(.enabled)    // или
enableNewFeature(true as FeatureFlag)  // но лучше:
let newFeature: FeatureFlag = true
enableNewFeature(newFeature)           // очень ясно
```

### 3. Реальные примеры из практики 2026 года

#### Пример 1: SwiftUI-стиль (padding, opacity и т.д.)

```swift
struct Padding: ExpressibleByBoolLiteral {
    let value: CGFloat
    init(booleanLiteral value: Bool) {
        self.value = value ? 16 : 0
    }
}

let padding: Padding = true   // → 16
let noPadding: Padding = false // → 0
```

#### Пример 2: Флаги конфигурации (очень популярно в 2025–2026)

```swift
struct LoggingLevel: ExpressibleByBoolLiteral {
    let isVerbose: Bool
    init(booleanLiteral value: Bool) {
        self.isVerbose = value
    }
}

let verboseLogging: LoggingLevel = true
let quietLogging: LoggingLevel = false
```

#### Пример 3: Разрешения / доступ

```swift
struct CameraPermission: ExpressibleByBoolLiteral, CustomStringConvertible {
    let granted: Bool
    init(booleanLiteral value: Bool) {
        self.granted = value
    }
    
    var description: String {
        granted ? "разрешено" : "запрещено"
    }
}

let camera: CameraPermission = true
print(camera) // "разрешено"
```

### 4. Как компилятор решает, какой тип использовать

```swift
let x = true           // → Bool

let y: FeatureFlag = true  // → FeatureFlag(booleanLiteral: true)

let z: Permission = true   // → Permission(booleanLiteral: true)
```

**Контекст** (тип переменной, аргумента функции, возвращаемого значения) **решает всё**.

### 5. Можно ли сделать так, чтобы `true` значило что-то другое?

**Нет.**  
`true` и `false` — это **встроенные литералы** типа [[Bool]]`.BooleanLiteralType`.  
Вы не меняете смысл `true`, вы лишь говорите компилятору:  
«когда увидишь `true` в контексте моего типа — вызови мой `init(booleanLiteral:)`».

### 6. Сравнение с другими ExpressibleBy… протоколами

| Литерал        | Протокол                           | Пример использования                    |
| -------------- | ---------------------------------- | --------------------------------------- |
| `true`/`false` | ExpressibleByBoolLiteral           | FeatureFlag, Permission, ToggleState    |
| [[nil]]        | [[ExpressibleByNilLiteral]]        | [[Optional]], custom optional-like типы |
| `42`           | [[ExpressibleByIntegerLiteral]]    | BigInt, CustomNumber                    |
| `"text"`       | [[ExpressibleByStringLiteral]]     | LocalizedString, [[URL]], Regex         |
| `[1, 2, 3]`    | [[ExpressibleByArrayLiteral]]      | CustomArray, OrderedSet                 |
| `["a": 1]`     | [[ExpressibleByDictionaryLiteral]] | CustomDict, CaseInsensitiveDict         |

### 7. Лучшие практики ExpressibleByBoolLiteral в 2026 году

- **Используйте**, когда хотите **улучшить читаемость** и **добавить семантику**  
- **Не заменяйте** `Bool` повсеместно — это антипаттерн  
- **Лучше всего работает** в DSL, конфигурациях, declarative [[API]]  
- **Называйте типы осмысленно**: `FeatureEnabled`, `PermissionGranted`, `DarkModeActive`  
- **Добавляйте ExpressibleByNilLiteral**, если тип может быть «выключен» через `nil`  
- **Swift 6 strict concurrency** — типы, реализующие протокол, должны быть `Sendable`  
- **Документируйте** — пиши комментарий «ExpressibleByBoolLiteral — позволяет писать `enabled = true` вместо `enabled = FeatureEnabled(true)`»

**Короткий девиз 2026**:
> `ExpressibleByBoolLiteral` — это когда ты превращаешь `true` / `false` в **смысл**, а не просто в `Bool`.  
> Используй его для **декларативного** и **читаемого** кода: флаги, разрешения, состояния, конфигурации.  
> Но **не злоупотребляй** — `Bool` всё ещё король для простой логики.
