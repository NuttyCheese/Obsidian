## 1. Что такое `@unchecked`

👉 `@unchecked` — это атрибут, который говорит компилятору:

> «Я сам гарантирую безопасность этого типа в многопоточности, даже если компилятор не может это проверить».

То есть:

- Обычно, чтобы тип был безопасен для передачи между потоками, он должен соответствовать `Sendable`.
    
- Если компилятор видит, что тип **не является потокобезопасным** (например, содержит ссылочные типы с изменяемыми свойствами), он не даст его автоматически сделать `Sendable`.
    
- В таких случаях мы можем написать `@unchecked Sendable`, чтобы **обойти проверку компилятора**.
    

⚠️ Но ответственность теперь лежит на нас. Если тип всё-таки не потокобезопасный → возможны гонки данных и баги.

---

## 2. Пример без `@unchecked`

```swift
final class Logger {
    var messages: [String] = []  // изменяемый массив
}
```

👉 Компилятор скажет:  
❌ `Logger` не может быть `Sendable`, так как это класс с изменяемым состоянием.

---

## 3. Пример с `@unchecked`

```swift
final class Logger: @unchecked Sendable {
    var messages: [String] = []
}
```

👉 Теперь компилятор не ругается, но вся ответственность на программисте:

- Если использовать `Logger` из разных task одновременно → будут гонки данных.
    
- Но если мы, например, гарантируем, что `Logger` всегда будет использоваться **только внутри одного actor** или через синхронизацию ([[NSLock]], [[DispatchQueue]]), то это допустимо.
    

---

## 4. Примеры от простого к сложному

### Пример 1. Структура (безопасный тип)

```swift
struct User: Sendable {
    let name: String
}
```

👉 [[String]] уже `Sendable`, значит `User` автоматически безопасный.

---

### Пример 2. Класс без `Sendable`

```swift
final class User {
    var name: String
    init(name: String) { self.name = name }
}
```

👉 ❌ Компилятор: класс — ссылочный тип, не потокобезопасный.

---

### Пример 3. Класс с `@unchecked Sendable`

```swift
final class User: @unchecked Sendable {
    var name: String
    init(name: String) { self.name = name }
}
```

👉 ✅ Теперь можно использовать в `Task.detached {}` или других местах, где требуется `Sendable`.

---

### Пример 4. Использование в [[Task]]

```swift
final class Logger: @unchecked Sendable {
    var text = ""
}

let logger = Logger()

Task.detached {
    // можно использовать logger, даже если он не потокобезопасный
    logger.text = "Hello"
    print(logger.text)
}
```

👉 ⚠️ Это будет работать, но если несколько Task будут менять `logger.text` одновременно — возможна гонка данных.

---

### Пример 5. Гарантия безопасности через actor

```swift
final class Logger: @unchecked Sendable {
    var messages: [String] = []
}

actor LogService {
    private let logger = Logger()

    func add(_ message: String) {
        logger.messages.append(message) // безопасно, т.к. доступ только внутри actor
    }

    func getAll() -> [String] {
        return logger.messages
    }
}
```

👉 Здесь `Logger` хоть и `@unchecked`, но его использование безопасно, потому что доступ к нему идёт через actor.

---

### Пример 6. Совместимость со старым [[API]]

```swift
// Старый Objective-C класс
class LegacyNetworkManager: NSObject { }

final class NetworkWrapper: @unchecked Sendable {
    let manager = LegacyNetworkManager()
}
```

👉 Иногда приходится использовать `@unchecked`, когда работаем с [[Objective-C]] кодом, который не знает о Swift Concurrency.

---

## 5. Когда использовать `@unchecked`

✅ Можно:

- Когда вы точно знаете, что тип будет использоваться **только в безопасном контексте** (через actor, lock, очередь).
    
- При обёртке над **старым кодом (Objective-C, C-библиотеки)**.
    
- Если API требует `Sendable`, а у вас нет возможности переписать всё полностью.
    

❌ Не стоит:

- Если тип реально используется конкурентно и вы не обеспечили синхронизацию.
    
- Как «быструю заплатку», чтобы убрать ошибку компилятора.
    

---

## 6. Итог

- `@unchecked` — это **обещание компилятору**, что тип `Sendable`, даже если он сам так не считает.
    
- Это мощный, но опасный инструмент: он убирает защиту компилятора, поэтому его стоит использовать осторожно.
    
- Обычно `@unchecked Sendable` используют для классов и старых [[API]].
    

---
