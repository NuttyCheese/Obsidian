**`Hashable`** — это протокол в стандартной библиотеке [[Swift]], который позволяет типу участвовать в **хэш-таблицах** ([[Set Collection]], ключи [[Dictionary]]).  
Он наследуется от `Equatable` и требует реализации метода `hash(into:)`.

### Основное требование протокола (2026)

```swift
public protocol Hashable : Equatable {
    func hash(into hasher: inout Hasher)
}
```

- `Equatable` обязателен → тип должен уметь сравниваться на `==`
- `hash(into:)` — метод, который добавляет все важные свойства в хэшер

### Когда тип обязан быть Hashable

| Коллекция / сценарий            | Требование к элементу / ключу | Что происходит без Hashable |
| ------------------------------- | ----------------------------- | --------------------------- |
| `Set<T>`                        | `T: Hashable`                 | Не скомпилируется           |
| `Dictionary<Key, Value>` (ключ) | `Key: Hashable`               | Не скомпилируется           |
| `Dictionary` с кастомным ключом | `Key: Hashable`               | —                           |
| `NSSet`, `NSOrderedSet` (редко) | `Hashable` + `NSObject`       | —                           |
| `Identifiable` в SwiftUI (id)   | `ID: Hashable`                | [[SwiftUI]] требует         |

### Автоматическая реализация (самый частый случай)

Swift **автоматически** синтезирует `Hashable` для:

- **структур** ([[struct]])  
- **перечислений** ([[enum]])  
если **все** их свойства / ассоциированные значения сами соответствуют `Hashable`.

```swift
// Автоматически Hashable
struct User: Hashable {
    let id: UUID
    let name: String
    let age: Int
    let isActive: Bool
}

// Автоматически Hashable
enum NetworkStatus: Hashable {
    case idle
    case loading(progress: Double)
    case success(statusCode: Int)
    case failure(error: Error)
}
```

**Правило**: если все поля/[[associated value]]s — `Hashable`, компилятор сам напишет `==` и `hash(into:)`.

### Ручная реализация (когда нужно управлять)

```swift
struct Employee: Hashable {
    let id: Int
    let name: String
    let department: String
    
    // Только id влияет на хэш и равенство
    func hash(into hasher: inout Hasher) {
        hasher.combine(id)
    }
    
    static func == (lhs: Employee, rhs: Employee) -> Bool {
        lhs.id == rhs.id
    }
}

let emp1 = Employee(id: 1, name: "Алекс", department: "iOS")
let emp2 = Employee(id: 1, name: "Александр", department: "Android")

print(emp1 == emp2)          // true (по id)
print(emp1.hashValue == emp2.hashValue) // true
```

**Самая частая ошибка** — забыть реализовать `==` согласованно с `hash(into:)`.  
Если `a == b`, то **обязательно** `a.hashValue == b.hashValue`.

### Самые популярные паттерны Hashable в 2026 году

#### 1. Уникальность по ID (очень часто)

```swift
struct Post: Hashable, Identifiable {
    let id: UUID
    let title: String
    let content: String
    let author: String
    
    func hash(into hasher: inout Hasher) {
        hasher.combine(id)
    }
    
    static func == (lhs: Post, rhs: Post) -> Bool {
        lhs.id == rhs.id
    }
}
```

#### 2. Hashable + [[Codable]] (модели [[API]])

```swift
struct Product: Codable, Hashable, Identifiable {
    let id: String
    let name: String
    let price: Decimal
    
    func hash(into hasher: inout Hasher) {
        hasher.combine(id)
    }
    
    static func == (lhs: Product, rhs: Product) -> Bool {
        lhs.id == rhs.id
    }
}
```

#### 3. Hashable enum с associated values

```swift
enum PaymentMethod: Hashable {
    case cash
    case card(last4: String)
    case applePay(token: String)
    
    func hash(into hasher: inout Hasher) {
        switch self {
        case .cash:
            hasher.combine(0)
        case .card(let last4):
            hasher.combine(1)
            hasher.combine(last4)
        case .applePay(let token):
            hasher.combine(2)
            hasher.combine(token)
        }
    }
    
    static func == (lhs: PaymentMethod, rhs: PaymentMethod) -> Bool {
        switch (lhs, rhs) {
        case (.cash, .cash): return true
        case let (.card(l), .card(r)): return l == r
        case let (.applePay(l), .applePay(r)): return l == r
        default: return false
        }
    }
}
```

### 4. Лучшие практики Hashable в Swift 2026

- **По умолчанию** — оставляй **автоматическую синтезировку** (быстрее и безопаснее)  
- Реализуй вручную **только** если нужно игнорировать какие-то поля (например, только по `id`)  
- **Согласованность** — если `a == b`, то **обязательно** `a.hashValue == b.hashValue`  
- **Не используй** изменяемые свойства в `hash(into:)` — это приведёт к багам в `Set`/`Dictionary`  
- **Swift 6 strict concurrency** — типы `Hashable` должны быть [[Sendable]] (enum и struct обычно ок)  
- **Документируйте** — пиши комментарий «Hashable — равенство и хэш только по id»

**Короткий девиз 2026**:
> `Hashable` — это когда ты хочешь, чтобы твой тип жил в `Set` и был ключом в `Dictionary`.  
> В 2026 году:  
> - оставляй синтез, если все поля важны  
> - пиши вручную, если важен только ID / ключ  
> - всегда держи `==` и `hash(into:)` согласованными  
> Это **основа** эффективных коллекций и уникальности в Swift.
