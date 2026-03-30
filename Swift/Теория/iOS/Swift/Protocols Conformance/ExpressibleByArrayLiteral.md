**`ExpressibleByArrayLiteral`** — это протокол в стандартной библиотеке [[Swift]], который позволяет типу быть инициализированным с помощью **литерала массива** (array literal), то есть синтаксиса `[элемент1, элемент2, …]`.

Если тип соответствует этому протоколу, вы можете писать:

```swift
let values: MyType = [1, 2, 3, 4]
```

вместо более громоздкого создания через обычный `init`.

### Основные требования протокола

```swift
public protocol ExpressibleByArrayLiteral {
    associatedtype ArrayLiteralElement
    init(arrayLiteral elements: ArrayLiteralElement...)
}
```

- `ArrayLiteralElement` — тип элементов, которые могут быть внутри литерала
- `init(arrayLiteral:)` — обязательный инициализатор, который принимает variadic-параметр (ноль или больше элементов)

### Самые известные типы, которые соответствуют протоколу

| Тип                                  | ArrayLiteralElement | Пример инициализации                            | Комментарий                     |
| ------------------------------------ | ------------------- | ----------------------------------------------- | ------------------------------- |
| [[Array]]`<T>`                       | `T`                 | `[1, 2, 3]`                                     | Самый очевидный случай          |
| [[Set Collection]]`<T>` (где `T:` [[Hashable]]) | `T`                 | `Set([1, 2, 3])` или просто `[1, 2, 3]` как Set | Удобный синтаксис с [[iOS]] 13+ |
| [[Dictionary]]`<Key, Value>`         | `(Key, Value)`      | `["a": 1, "b": 2]`                              | Кортежи (key, value)            |
| `ContiguousArray<T>`                 | `T`                 | Редко используется напрямую                     | —                               |
| `ArraySlice<T>`                      | `T`                 | Получается при срезе массива                    | —                               |

### Самые популярные пользовательские реализации (2025–2026)

#### 1. Простой контейнер (самый частый учебный пример)

```swift
struct PointCloud: ExpressibleByArrayLiteral {
    private var points: [(x: Double, y: Double, z: Double)] = []
    
    init(arrayLiteral elements: (x: Double, y: Double, z: Double)...) {
        self.points = Array(elements)
    }
    
    var count: Int { points.count }
}

let cloud: PointCloud = [
    (1, 2, 3),
    (4, 5, 6),
    (7, 8, 9)
]

print(cloud.count) // 3
```

#### 2. Более практичный пример — коллекция идентификаторов

```swift
struct UserIDs: ExpressibleByArrayLiteral, ExpressibleByStringLiteral {
    private var ids: Set<UUID> = []
    
    init(arrayLiteral elements: UUID...) {
        self.ids = Set(elements)
    }
    
    // Бонус: поддержка строковых литералов
    init(stringLiteral value: String) {
        if let uuid = UUID(uuidString: value) {
            self.ids = [uuid]
        }
    }
    
    init(arrayLiteral elements: String...) {
        self.ids = Set(elements.compactMap(UUID.init(uuidString:)))
    }
}

// Использование
let team: UserIDs = [
    "550e8400-e29b-41d4-a716-446655440000",
    UUID(),
    UUID()
]

let single: UserIDs = "550e8400-e29b-41d4-a716-446655440000"
```

#### 3. Очень популярный паттерн — Result с массивом ошибок

```swift
enum ValidationResult: ExpressibleByArrayLiteral {
    case valid
    case invalid([String]) // массив сообщений об ошибках
    
    init(arrayLiteral elements: String...) {
        if elements.isEmpty {
            self = .valid
        } else {
            self = .invalid(Array(elements))
        }
    }
}

// Использование
let result: ValidationResult = ["Имя слишком короткое", "Email некорректный"]
// или
let ok: ValidationResult = []           // → .valid
```

### Лучшие практики ExpressibleByArrayLiteral в Swift 2026

- **Используйте**, когда хотите дать типу **красивый и естественный синтаксис** создания через `[ … ]`  
- **Чаще всего** реализуют для:
  - коллекций с уникальными элементами (Set-подобные)
  - структур-обёрток над массивами
  - DSL (domain-specific languages)
  - конфигураций
  - результатов валидации
- **Не злоупотребляйте** — если тип сложный и требует много логики при создании → лучше обычный `init`  
- **Комбинируйте** с другими Expressible-протоколами:
  - [[ExpressibleByStringLiteral]]
  - [[ExpressibleByDictionaryLiteral]]
  - [[ExpressibleByIntegerLiteral]]
- **[[Swift]] 6 strict concurrency** — тип, реализующий протокол, должен быть [[Sendable]], если используется в акторах  
- **Документируйте** — пишите комментарий «ExpressibleByArrayLiteral — позволяет инициализацию через [UUID(), UUID()]»

**Короткий девиз 2026**:
> `ExpressibleByArrayLiteral` — это когда вы хотите, чтобы ваш тип можно было создавать так же естественно, как обычный массив: `[1, 2, 3]`.  
> В 2026 году:  
> - используйте для красивого [[API]] коллекций, конфигураций, валидаторов  
> - чаще всего реализуют с `Set`, `Array` или кастомными контейнерами  
> - комбинируйте с другими Expressible*-протоколами  
> Это **мощный инструмент** для выразительного и declarative кода.
