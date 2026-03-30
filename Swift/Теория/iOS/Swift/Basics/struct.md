**`struct`** — это **тип значения** ([[value type]]) в [[Swift]], который объединяет свойства, методы, вычисляемые свойства, инициализаторы и может соответствовать протоколам.

Это **самый часто используемый** тип в современном Swift-коде (2025–2026), потому что он:
- безопасен (копируется при передаче)
- предсказуем (нет скрытых ссылок)
- эффективен по памяти ([[Copy-on-Write]] для коллекций)
- отлично работает с многопоточностью (Swift 6 strict concurrency)

### 1. Ключевые отличия struct от class (2026 взгляд)

| Характеристика               | struct (Value Type)                                     | [[class]] ([[Reference Type]])          | Когда выбирать struct         |
| ---------------------------- | ------------------------------------------------------- | --------------------------------------- | ----------------------------- |
| Семантика передачи           | Копируется (value semantics)                            | Передаётся ссылка (reference semantics) | Почти всегда (модели, данные) |
| Наследование                 | Нет                                                     | Да                                      | Не нужно наследование         |
| Mutating методы              | Требуется `mutating` для изменения свойств              | Не требуется                            | —                             |
| Deinit                       | Нет                                                     | Есть                                    | Не нужен                      |
| ARC (счётчик ссылок)         | Нет                                                     | Есть                                    | Нет [[retain cycle]]s         |
| Copy-on-Write                | Да (для [[Array]], [[String]], [[Set Collection]], [[Dictionary]]) | Нет                                     | Экономия памяти               |
| Thread-safety (по умолчанию) | Полностью безопасен                                     | Требует осторожности                    | Swift 6+ преимущество         |

**Золотое правило 2026**:  
«Если не нужен наследование и shared mutable state — используй `struct` (или `enum`).»

### 2. Самые популярные паттерны struct в 2026 году

#### 2.1 Модель данных (Codable, Identifiable, Equatable, Hashable)

```swift
struct User: Codable, Identifiable, Equatable, Hashable {
    let id: UUID
    let name: String
    let email: String
    let age: Int
    let isActive: Bool
    
    // Автоматически синтезируется Equatable и Hashable
}
```

#### 2.2 Конфигурация / Value Object

```swift
struct AppConfig {
    let apiBaseURL: URL
    let maxRetryCount: Int = 3
    let timeout: TimeInterval = 30
    
    static let `default` = AppConfig(apiBaseURL: URL(string: "https://api.example.com")!)
}
```

#### 2.3 Computed property + mutating method

```swift
struct Counter {
    private(set) var count = 0
    
    var isZero: Bool {
        count == 0
    }
    
    mutating func increment(by step: Int = 1) {
        count += step
    }
    
    mutating func reset() {
        count = 0
    }
}
```

#### 2.4 Struct с вложенными типами

```swift
struct Order {
    struct Item {
        let productID: String
        let quantity: Int
        let price: Decimal
    }
    
    let id: UUID
    var items: [Item]
    var total: Decimal {
        items.reduce(0) { $0 + $1.price * Decimal($1.quantity) }
    }
}
```

### 3. Лучшие практики struct в Swift 2026

- **Делай все свойства `let`**, если они не должны меняться после создания  
- **Используй `private(set)`**, когда нужно читать свойство снаружи, но писать — только внутри  
- **Добавляй `mutating` методы** только для изменения состояния  
- **Соответствуй протоколам** автоматически ([[Equatable]], [[Hashable]], [[Codable]]) — Swift сам синтезирует  
- **Используй `static let` / `static func`** для фабричных методов и констант  
- **Не делай struct слишком большим** — если больше 5–7 свойств, подумай о вложенных struct или отдельные типы  
- **Swift 6 strict concurrency** — struct полностью `Sendable`, если все свойства `Sendable`  
- **Документируйте** — пиши комментарий «struct User — immutable модель пользователя из API»

**Короткий девиз 2026**:
> `struct` — это **безопасный, копируемый, предсказуемый** контейнер для данных.  
> В 2026 году:  
> - используй `struct` по умолчанию для всех моделей, конфигураций, value objects  
> - все свойства `let`, если не нужна мутация  
> - `mutating` методы — только для изменения состояния  
> - протоколы (Codable, Equatable, Hashable) — синтезируй автоматически  
> Это **основа** современного Swift-кода без утечек памяти и сюрпризов.
