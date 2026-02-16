**NSNull** — это специальный класс из **Foundation**, который используется в [[Objective-C]] и Swift для представления **отсутствия значения** (null) в ситуациях, где **[[nil]]** недопустим.

В 2026 году (Swift 6+, iOS 18+, macOS 15+) **NSNull** всё ещё активно применяется, но его использование в чистом Swift-коде **минимально** и почти всегда связано с **Objective-C совместимостью** или legacy-API.

### Ключевые отличия NSNull от nil (2026 актуальные)

| Характеристика                       | nil (Swift)                         | NSNull (Objective-C / [[Foundation]])                     | Когда использовать                     |
| ------------------------------------ | ----------------------------------- | --------------------------------------------------------- | -------------------------------------- |
| Тип                                  | `nil` ([[Optional]].none)           | Singleton-объект класса `NSNull`                          | Когда [[API]] требует объект, а не nil |
| Может быть в коллекциях              | Нет (Array не может содержать nil)  | Да ([[NSArray]], [[NSDictionary]] могут содержать NSNull) | В ObjC-коллекциях                      |
| Bridging в Swift                     | —                                   | `NSNull()` → `NSNull` (не Optional)                       | При получении из ObjC                  |
| [[Codable]] / [[JSON]]               | nil → отсутствует ключ или null     | NSNull → JSON null                                        | JSON-парсинг из ObjC                   |
| [[Swift Concurrency]] ([[Sendable]]) | nil — Sendable                      | NSNull — Sendable                                         | Оба безопасны                          |
| Использование в новом коде           | Почти 100% случаев (nil / Optional) | Только в ObjC-[[API]] и legacy                            | Минимально                             |

### Когда NSNull всё ещё нужен в 2026 году (реальные сценарии)

| Сценарий                                                                                                                          | Почему NSNull используется                     | Рекомендация / альтернатива                             |
| --------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------- | ------------------------------------------------------- |
| Получение данных из **[[Objective-C]] [[API]]** ([[NSDictionary]], [[NSArray]], [[UserDefaults]], [[Core Data]], [[KVC]]/[[KVO]]) | Многие старые API используют NSNull вместо nil | Приводи сразу к Optional: `value as? String ?? nil`     |
| **JSONSerialization** (JSONSerialization.jsonObject)                                                                              | Возвращает NSNull для null-значений в JSON     | Используй JSONDecoder с `.convertFromNull` или фильтруй |
| **UserDefaults** (dictionaryRepresentation)                                                                                       | Может содержать NSNull                         | Переходи на `Codable` + JSON в UserDefaults             |
| **KVC / KVO** (value(forKey:))                                                                                                    | Возвращает NSNull для null-значений            | Combine / Observation (Swift 5.9+)                      |
| **[[Core Data]]** (transformable attributes)                                                                                      | Иногда хранит NSNull                           | Используй `Codable` атрибуты                            |
| **Legacy-код** / поддержка iOS 12–14                                                                                              | Совместимость                                  | Миграция на [[Optional]] / [[nil]]                      |

### Самые популярные и рекомендуемые паттерны NSNull → Swift в 2026

#### Паттерн 1 — Безопасное приведение NSNull к Optional

```swift
let nsDict = someObjCObject.dictionaryValue() as NSDictionary

// Самый безопасный и современный способ
let name = nsDict["name"] as? String
let age = nsDict["age"] as? Int

// Или одним проходом (фильтруем NSNull)
let cleanDict = nsDict.reduce(into: [String: Any]()) { result, pair in
    if pair.value !== NSNull() {
        if let key = pair.key as? String {
            result[key] = pair.value
        }
    }
}
```

#### Паттерн 2 — JSONSerialization и обработка NSNull

```swift
let jsonData: Data = ... // от сервера

do {
    let object = try JSONSerialization.jsonObject(with: jsonData)
    
    // NSNull в словаре / массиве
    if let dict = object as? [String: Any] {
        let name = dict["name"] as? String ?? "Unknown"  // NSNull → nil → default
    }
} catch {
    print("JSON error: \(error)")
}
```

**Лучшая практика 2026** — не используй JSONSerialization.  
Переходи на **JSONDecoder**:

```swift
struct User: Codable {
    let name: String?
    let age: Int?
    
    // NSNull автоматически преобразуется в nil
}
```

#### Паттерн 3 — Современный стиль: только Optional и nil (без NSNull)

```swift
func parseResponse(_ dict: [String: Any]) -> String? {
    // NSNull автоматически преобразуется в nil при as?
    return dict["message"] as? String
}
```

### Лучшие практики NSNull в Swift 2026

- **Избегай NSNull в чистом Swift-коде** — используй `nil` / Optional  
- **При получении NSDictionary/NSArray из ObjC** — сразу фильтруй NSNull через `as?` или `compactMap`  
- **JSONSerialization** → заменяй на **JSONDecoder** (автоматически обрабатывает null → nil)  
- **Swift 6 strict concurrency** — `NSNull` **Sendable**, но передавай через `@MainActor` или копируй в Swift-структуры  
- **Тестирование** — моки NSNull через `NSNull()`  
- **Документируйте** — пиши комментарий «NSDictionary из Objective-C API — фильтруем NSNull»

**Короткий девиз 2026**:
> «NSNull в 2026 году — это когда Objective-C API хочет сказать «здесь нет значения», но не может использовать nil.  
> В чистом Swift почти всё заменено на nil / Optional.  
> Главное правило: как только получил NSNull — сразу преобразуй в nil и забудь про него.»
