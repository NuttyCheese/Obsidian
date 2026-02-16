**Sendable** — это протокол в Swift, который указывает, что тип **безопасно передавать** между разными задачами (tasks) и актёрами (actors) в модели конкурентности Swift.

С появлением **Swift Concurrency** (async/await, Task, actor) в Swift 5.5 и особенно с **strict concurrency checking** в Swift 6 (2024–2026), Sendable стал одним из самых важных и часто встречающихся протоколов в современном Swift-коде.

### Что именно значит «Sendable»

Тип считается Sendable, если его значение можно безопасно передать из одной конкурентной задачи в другую **без риска data race**.

Основные требования к типу, чтобы он был Sendable:

1. **Immutable** (неизменяемый) — либо полностью const, либо мутация происходит только внутри изолированного контекста (actor)  
2. **Value semantics** — копирование не делит состояние (как struct, enum, String, Array и т.д.)  
3. **Все свойства Sendable** — если тип содержит другие типы, они тоже должны быть Sendable  
4. **Нет сильных ссылок** на mutable shared state вне изоляции

### Типы, которые автоматически Sendable (2026)

| Категория                  | Примеры типов, которые Sendable по умолчанию | Примечание |
|----------------------------|-----------------------------------------------|------------|
| Все value types без мутабельных ссылок | struct, enum, tuple                           | Если все свойства Sendable |
| Основные типы Foundation   | String, Data, Date, UUID, URL, Decimal        | Полностью Sendable |
| Коллекции                  | Array<T>, Dictionary<Key, Value>, Set<T>      | Если T / Key / Value Sendable |
| Числовые типы              | Int, UInt, Float, Double, Float16, Bool       | Sendable всегда |
| Optional                   | T?                                            | Sendable, если T Sendable |
| Actor                      | actor MyActor                                 | Sendable всегда (изоляция) |
| @Sendable closure          | () -> Void, () async throws -> Void           | Явно помеченные замыкания |

### Типы, которые НЕ Sendable по умолчанию (и почему)

| Тип / Класс                | Почему не Sendable                              | Как сделать Sendable |
|----------------------------|-------------------------------------------------|----------------------|
| class (обычный)            | Reference semantics + mutable shared state      | `final class` + `Sendable` (если immutable) или `@unchecked Sendable` |
| Actor (но свойства)        | Сам actor Sendable, но его свойства — нет       | Доступ только через изолированные методы |
| any Protocol (existential) | Неизвестный тип внутри → компилятор не уверен  | Используй `some Protocol` или primary associated types |
| NSMutableArray / NSDictionary | Mutable reference type                          | Копируй в [AnyHashable: Any] или Array/Dictionary |
| UIView / UIViewController  | UI-классы, привязаны к main thread              | Используй @MainActor |

### Как пометить тип как Sendable

1. **Автоматически** — если все свойства Sendable и тип value semantics  
   → struct / enum / actor почти всегда Sendable

2. **Явно** — с помощью conformance

```swift
final class Logger: @unchecked Sendable {
    static let shared = Logger()
    private init() {}
    
    func log(_ message: String) {
        print(message)
    }
}
```

`@unchecked Sendable` — когда компилятор не может доказать безопасность, но ты уверен вручную (чаще всего для singleton’ов, final class без мутабельного состояния).

3. **@Sendable** для замыканий

```swift
func runInBackground(_ work: @Sendable @escaping () -> Void) {
    Task.detached {
        work()
    }
}
```

### Лучшие практики Sendable в Swift 2026

- **Предпочитай value semantics** (struct, enum) — они почти всегда Sendable  
- **actor** — для mutable shared state (заменил большинство class + lock)  
- **@MainActor** — для всего UI-related (UIViewController, ViewModel) — автоматически Sendable в main-изоляции  
- **final class** + immutable свойства — можно пометить Sendable  
- **@unchecked Sendable** — используй осторожно, только когда уверен в безопасности (singleton, cache без мутации)  
- **any Protocol** — избегай в Task / actor-передачах → предпочитай `some Protocol` или generics  
- **Swift 6 strict concurrency checking** — включает **полную проверку Sendable** → почти весь код придётся пометить или переписать  
- **Тестирование** — проверяй data race через Thread Sanitizer + stress-тесты  
- **Документируйте** — пиши комментарий «@unchecked Sendable — singleton без mutable state»

**Короткий девиз 2026**:
> «Sendable — это когда ты говоришь компилятору: «этот тип можно безопасно передать между задачами и актёрами без data race».  
> В Swift 6+ почти всё, что ты пишешь в конкурентном коде, должно быть Sendable.  
> Основное правило: struct / enum / actor — почти всегда Sendable, class — только если final + immutable или @unchecked Sendable.»

Удачи с безопасным и строгим конкурентным кодом в Swift! ⚡