**`@isolated(any)`** — это **атрибут параметра функции**, который говорит компилятору:

> «Этот параметр — актёр ([[actor]]), и **вся функция должна выполняться в изоляции именно этого актёра**».

Другими словами:

- функция **наследует изоляцию** от переданного экземпляра actor  
- внутри функции можно **прямо** обращаться к изолированным свойствам и методам этого [[actor]]-а **без [[await]]**  
- компилятор **гарантирует**, что тело функции будет выполняться **внутри контекста переданного актёра**

Это один из самых мощных и современных инструментов работы с изоляцией в [[Swift Concurrency]] (введён в Swift 5.7–5.9, значительно улучшен в [[Swift]] 6).

**Коротко**:
- без `@isolated(any)` → нужно `await actor.method()`  
- с `@isolated(any)` → можно `actor.method()` напрямую, как будто мы уже внутри actor

### 2. Синтаксис и обязательные условия

```swift
func doSomething(with actor: isolated(any) SomeActor) {
    // внутри функции можно обращаться к actor.value напрямую
    actor.value += 1
    actor.someMethod()  // без await!
}
```

**Обязательные условия (2026)**:

- параметр **должен быть типа actor** (или протокола, который наследуется от `Actor`)  
- атрибут `@isolated(any)` ставится **только перед типом параметра**  
- функция **не может быть** помечена `@MainActor` или другим глобальным актёром одновременно (конфликт изоляции)  
- если функция асинхронная (`async`), то вызов всё равно будет **синхронным** внутри изоляции (но сам вызов функции может требовать `await`)

### 3. Самые популярные и правильные шаблоны использования в 2026

#### Шаблон 1 — Универсальная функция для работы с любым actor

```swift
func clearStorage<S: Actor>(_ storage: isolated(any) S) where S: StorageProtocol {
    storage.items.removeAll()
    storage.lastCleared = Date()
}

actor UserStorage: StorageProtocol {
    var items: [String] = []
    var lastCleared: Date?
}

actor OrderStorage: StorageProtocol {
    var items: [String] = []
    var lastCleared: Date?
}

let userStorage = UserStorage()
await clearStorage(userStorage)  // работает без await внутри

let orderStorage = OrderStorage()
await clearStorage(orderStorage)
```

#### Шаблон 2 — Работа с состоянием без [[await]]

```swift
actor Counter {
    var value = 0
    func increment() { value += 1 }
}

func batchIncrement(_ counter: isolated(any) Counter, times: Int) {
    for _ in 0..<times {
        counter.increment()     // без await!
        counter.value += 1      // прямой доступ
    }
}

let counter = Counter()
Task {
    await batchIncrement(counter, times: 1000)
}
```

#### Шаблон 3 — Универсальная обработка ошибок внутри actor

```swift
protocol SafeActor: Actor {
    associatedtype Value
    var value: Value { get set }
}

func safeUpdate<A: SafeActor>(_ actor: isolated(any) A, _ transform: (inout A.Value) throws -> Void) rethrows {
    try transform(&actor.value)
}

actor SafeInt: SafeActor {
    var value = 0
}

let safe = SafeInt()
try await safeUpdate(safe) { $0 += 42 }
```

#### Шаблон 4 — Работа с несколькими actor одновременно (с осторожностью)

```swift
actor UserDB {
    var users: [String] = []
}

actor OrderDB {
    var orders: [String] = []
}

func migrateUsers(to orderDB: isolated(any) OrderDB, from userDB: isolated(any) UserDB) {
    // Оба actor доступны без await
    let users = userDB.users
    orderDB.orders.append(contentsOf: users)
}
```

**Внимание**: вызов `migrateUsers` требует `await`, но внутри функции **нет await** — всё происходит в изоляции переданных актёров.

### 4. Типичные ошибки и ловушки 2026 года

| Ошибка                                      | Последствия                              | Как избежать |
|---------------------------------------------|------------------------------------------|--------------|
| Поставить `@isolated(any)` на не-actor параметр | Ошибка компиляции                        | Параметр должен быть actor-типа |
| Вызывать функцию без `await` извне          | Ошибка компиляции                        | Функция с `@isolated(any)` требует `await` при вызове |
| Думать, что функция не требует `await`      | Ошибка компиляции при вызове             | Вызов всегда через `await` |
| Использовать с `@MainActor` одновременно    | Конфликт изоляции                        | Нельзя комбинировать `@MainActor` и `@isolated(any)` |
| Передавать mutable значение без Sendable    | Ошибка strict concurrency                | Всё передаваемое должно быть Sendable |

### 5. @isolated(any) vs другие механизмы изоляции (2026 сравнение)

| Механизм                  | Изоляция наследуется от параметра | Требуется await при вызове | Сложность | Рекомендация 2026 | Когда использовать                 |
| ------------------------- | --------------------------------- | -------------------------- | --------- | ----------------- | ---------------------------------- |
| `@isolated(any)`          | Да                                | Да                         | Средняя   | Современный       | Универсальные функции для actor-ов |
| [[@MainActor]]            | Нет (глобальный)                  | Нет (если уже на [[main]]) | Низкая    | Для UI            | Всё, что связано с UI              |
| обычный [[actor]]         | Да (экземплярный)                 | Да                         | Низкая    | Основной выбор    | Локальное состояние                |
| [[@_inheritActorContext]] | Да (внутренний)                   | Нет                        | Высокая   | Legacy / обходной | Только если нет альтернативы       |

**Вывод 2026**:
- `@isolated(any)` — **самый современный и рекомендуемый** способ писать универсальные функции, работающие внутри произвольного actor-а  
- Он **значительно чище** и **безопаснее**, чем `@_inheritActorContext`  
- В 95% случаев заменяет старые костыли и делает код декларативным

### 6. Лучшие практики 2026 года

- **Используйте `@isolated(any)`** для всех универсальных функций, которые работают с actor-ами  
- **Всегда помечать** параметр как `isolated(any) ActorType`  
- **Вызов** такой функции **всегда** через `await`  
- **Не комбинировать** с `@MainActor` на одной функции  
- **Для UI** — предпочитайте `@MainActor`  
- **Тестирование** — легко мокать через протоколы и `isolated(any)`  
- **Swift 6 strict concurrency** — полностью поддерживается и очень полезен

**Короткий девиз 2026**:
> «@isolated(any) — это когда ты хочешь сказать функции: «работай внутри того actor-а, который мне передали».  
> Это один из самых красивых и мощных инструментов Swift Concurrency 2025–2026 годов.»
