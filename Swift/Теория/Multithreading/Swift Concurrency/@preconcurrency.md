## 1. Что такое `@preconcurrency`

👉 `@preconcurrency` — это **атрибут для аннотаций типов и протоколов**, который говорит компилятору:

> "Этот код был написан **до появления Concurrency** в [[Swift]]. Считай его _безопасным_ (или хотя бы не мешай компиляцией), даже если он не использует акторов и изоляцию".

Другими словами:

- Он помогает **мигрировать старые API** (особенно из Objective-C или C-библиотек) в мир Swift Concurrency.
    
- Убирает лишние `Sendable`/`@MainActor`/actor-изоляцию ошибки при работе с таким кодом.
    

---

## 2. Где используется

1. В стандартной библиотеке Swift (`Foundation`, `Dispatch`, `UIKit`).
    
2. В ваших [[API]], если вы пишете **библиотеку, совместимую со Swift 5.5+**, но часть кода ещё не concurrency-safe.
    

---

## 3. Пример из Foundation

В `Foundation` есть, например, `NotificationCenter`. Его API появилось задолго до Swift Concurrency.

Без `@preconcurrency` компилятор мог бы жаловаться:

```swift
let center = NotificationCenter.default
Task {
    // ❌ warning: reference to non-Sendable type 'Notification.Name' in a concurrency context
    center.addObserver(forName: .NSSystemClockDidChange, object: nil, queue: nil) { _ in
        print("Clock changed")
    }
}
```

Но Apple пометила [[NotificationCenter]] и часть его [[API]] как `@preconcurrency`.  
Это значит: «Да, этот код старый, мы разрешаем использовать его в новых concurrency-контекстах».

---

## 4. Ваши примеры

### Пример 1. Старый протокол

```swift
@preconcurrency protocol LegacyDelegate: AnyObject {
    func didUpdate()
}
```

👉 Теперь этот протокол можно использовать в actor или [[Task]], даже если он не `Sendable`.

---

### Пример 2. Класс, не готовый к Concurrency

```swift
@preconcurrency class LegacyManager {
    func doWork() { print("Работаю по старинке") }
}
```

---

### Пример 3. Использование в actor

```swift
actor Worker {
    let manager = LegacyManager()

    func run() {
        manager.doWork() // ✅ не требует Sendable
    }
}
```

---

### Пример 4. С протоколом без `Sendable`

```swift
@preconcurrency protocol LegacyProtocol {
    func process(data: Data)
}

actor Processor {
    var delegate: LegacyProtocol?

    func start() {
        delegate?.process(data: Data())
    }
}
```

👉 Без `@preconcurrency` компилятор мог бы ругаться, что протокол не `Sendable`.

---

### Пример 5. Для старого API библиотеки

```swift
@preconcurrency class OldNetworkClient {
    func fetch(_ url: String, completion: @escaping (String) -> Void) {
        completion("Result from \(url)")
    }
}

actor Service {
    let client = OldNetworkClient()

    func load() async {
        client.fetch("http://example.com") { result in
            print("Загружено:", result)
        }
    }
}
```

---

## 5. Когда использовать `@preconcurrency`

- Если у вас есть **старый код (или сторонняя библиотека)**, написанный до [[Swift]] Concurrency, и компилятор начинает требовать `Sendable`/`@MainActor` там, где вы этого не можете добавить.
    
- При написании **framework’ов или SDK**, чтобы сохранить обратную совместимость.
    
- Когда вы хотите временно подавить concurrency-ошибки, но **позже планируете переписать API с нормальной поддержкой actor/Sendable**.
    

---

## 6. Итоги

- `@preconcurrency` = "этот код написан до concurrency, не ругайся сильно".
    
- Помогает **обходить строгие правила Swift Concurrency** для старого кода.
    
- Используется в стандартной библиотеке ([[UIKit]], Foundation).
    
- В своём проекте применять стоит **только если нужно поддерживать legacy API**, а не как замену нормальной изоляции.
    

---
