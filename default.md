**Default** в [[Swift]] — это ключевое слово, которое используется в нескольких разных контекстах. Вот полный и актуальный (2026 год) обзор всех основных случаев использования `default` в современном Swift.

### 1. `default` в [[switch]]-выражении (самый частый случай)

`default` — это ветка, которая выполняется, когда ни один из явных `case` не подошёл.

```swift
enum Direction {
    case north, south, east, west
}

func describe(_ direction: Direction) -> String {
    switch direction {
    case .north: return "Вверх"
    case .south: return "Вниз"
    case .east:  return "Вправо"
    case .west:  return "Влево"
    // default обязателен, если enum не исчерпывающий
    default:     return "Неизвестное направление"
    }
}
```

**Важные нюансы 2026 года**:

- Если `switch` **исчерпывающий** (покрывает все кейсы enum / [[Bool]]), `default` **не нужен** и компилятор выдаст предупреждение  
- Если не исчерпывающий → `default` **обязателен**, иначе ошибка компиляции  
- В Swift 5.9+ можно использовать `#warning` или `#error` вместо `default` для документации:

```swift
switch direction {
case .north: return "Вверх"
// ...
@unknown default:
    #warning("Добавлен новый кейс Direction — обновите switch")
    return "Неизвестно"
}
```

### 2. `default` в `@Observable` / `@ObservableState` ([[TCA]] / Swift 6+)

В The Composable Architecture (TCA) и `@Observable` часто используется `default` для значений по умолчанию в `State`.

```swift
@ObservableState
struct ProfileState {
    var name: String = ""
    var age: Int? = nil
    var isLoading = false
    
    // default-значения для новых полей при миграции состояния
    var theme: Theme = .system  // default
}
```

### 3. `default` в параметрах функций (default arguments)

```swift
func fetchUsers(page: Int = 1, limit: Int = 20) async throws -> [User] {
    // ...
}

// Вызовы
let users1 = try await fetchUsers()           // page=1, limit=20
let users2 = try await fetchUsers(page: 3)    // page=3, limit=20
```

**Современный стиль 2026**:
- Используй **default-аргументы** вместо перегрузок функций  
- Делай параметры с дефолтом **последними** в списке

### 4. `default` в [[Codable]] / [[Decodable]] (default value при декодировании)

```swift
struct User: Codable {
    let name: String
    let age: Int?
    let isActive: Bool = true  // default, если поле отсутствует в JSON
    
    enum CodingKeys: String, CodingKey {
        case name, age
        case isActive = "active"
    }
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        name = try container.decode(String.self, forKey: .name)
        age = try container.decodeIfPresent(Int.self, forKey: .age)
        
        // Если ключа нет — берём default
        isActive = try container.decodeIfPresent(Bool.self, forKey: .isActive) ?? true
    }
}
```

**Лучшая практика 2026** — используй property wrapper `@Default` из swift-dependencies или свой собственный:

```swift
@propertyWrapper
struct Default<T: Decodable & ExpressibleByLiteral> {
    var wrappedValue: T
    
    init(from decoder: Decoder) throws {
        let container = try decoder.singleValueContainer()
        wrappedValue = (try? container.decode(T.self)) ?? T()
    }
}
```

### 5. `default` в `switch` с `fallthrough` (редко, но полезно)

```swift
switch statusCode {
case 200...299:
    print("Success")
case 400...499:
    print("Client error")
    fallthrough  // падает в default
default:
    print("Неизвестный / серверный код: \(statusCode)")
}
```

### 6. Лучшие практики использования `default` в Swift 2026

- **switch** — всегда добавляй `@unknown default` для future-proof кода с enum  
- **Параметры функций** — ставь default-значения в конец списка  
- **Codable** — используй `decodeIfPresent` + `?? defaultValue` или property wrapper  
- **@Observable / TCA** — явно задавай default-значения в State  
- **Swift 6 strict concurrency** — `default` не влияет на конкурентность, но следи за изоляцией  
- **Не злоупотребляй** — слишком много default-значений делает код менее явным

**Короткий девиз 2026**:
> «default в Swift — это когда ты говоришь компилятору: «если ничего не подошло — сделай вот это».  
> Самые частые места: switch, параметры функций, Codable, State в TCA.  
> Используй его для ясности и безопасности, но не прячь важную логику за default.»
