## 1. Что это такое?

`@_inheritActorContext` — **атрибут компилятора** (заметка: подчёркивание `_` говорит, что это _неофициальный / внутренний_ [[API]], Apple его не документирует и может изменить или удалить в будущем).

👉 Его основная задача — **указать компилятору, что функция или замыкание должно выполняться в том же actor-контексте, где оно было вызвано**.

Обычно в [[Swift]] Concurrency:

- если мы пишем `Task { ... }` — это создаёт _новый_ [[Task]], который может выполняться в произвольном потоке.
    
- если мы пишем `await` внутри actor, то всё остаётся в контексте [[actor]], пока мы не "выскочим" в отдельный task.
    

А вот `@_inheritActorContext` заставляет _сохранить текущий контекст actor_ при передаче функций/замыканий.

---

## 2. Как это работает?

- Обычно, если вы передаёте замыкание в `Task.init`, оно запускается **без привязки к actor**, даже если вы вызвали его изнутри `MainActor`.
    
- Но если добавить `@_inheritActorContext`, то компилятор постарается **не терять actor isolation** и продолжить выполнение внутри того же actor.
    

⚠️ Но: так как это _internal feature_, использовать её нужно осторожно — Apple может заменить это чем-то вроде `@MainActor(unsafe)` или других будущих атрибутов.

---

## 3. Применение в коде

Без `@_inheritActorContext`:

```swift
import Foundation

actor Counter {
    private var value = 0

    func increment() {
        Task {
            // ⚠️ это выполняется вне actor
            value += 1 // ❌ Ошибка: actor-isolated property 'value' can only be mutated within the actor
        }
    }
}
```

С `@_inheritActorContext`:

```swift
import Foundation

actor Counter {
    private var value = 0

    func increment() {
        Task @_inheritActorContext {
            // ✅ сохраняем контекст actor
            value += 1
        }
    }
}
```

---

## 4. Примеры

### Пример 1. С MainActor

```swift
import SwiftUI

@MainActor
class ViewModel: ObservableObject {
    @Published var text: String = "Старт"

    func updateText() {
        Task @_inheritActorContext {
            // Будет выполнено на MainActor, UI обновится безопасно
            text = "Обновлено!"
        }
    }
}
```

---

### Пример 2. Actor + Task

```swift
actor Storage {
    private var items: [String] = []

    func add(item: String) {
        Task @_inheritActorContext {
            items.append(item) // безопасно
        }
    }

    func getAll() -> [String] {
        return items
    }
}
```

---

### Пример 3. Сравнение с обычным `Task`

```swift
actor Logger {
    func log(_ message: String) {
        Task {
            // ❌ здесь потеряем actor context
            // print(message) можно, но доступ к actor-состоянию запрещён
        }

        Task @_inheritActorContext {
            // ✅ здесь context сохраняется
            print("Actor safe log:", message)
        }
    }
}
```

---

### Пример 4. Передача в функцию

```swift
func doWork(_ operation: @escaping () async -> Void) {
    Task {
        await operation()
    }
}

actor Worker {
    private var counter = 0

    func start() {
        doWork {
            // ⚠️ без @_inheritActorContext — ошибка при доступе к counter
            counter += 1
        }
    }

    func startSafe() {
        doWork { @_inheritActorContext in
            // ✅ контекст actor сохраняется
            counter += 1
        }
    }
}
```

---

### Пример 5. UI обновление из Actor

```swift
@MainActor
class AppState: ObservableObject {
    @Published var status: String = "Инициализация..."

    func updateStatus() {
        Task @_inheritActorContext {
            // остаёмся на MainActor
            status = "Загружено ✅"
        }
    }
}
```

---

## 5. Выводы

- `@_inheritActorContext` — это **внутренний инструмент Swift**, позволяющий **сохранять actor-контекст** при передаче задач.
    
- Полезно для случаев, когда вы хотите, чтобы `Task` **не "выпрыгивал" из actor**.
    
- Использовать можно, но с осторожностью, так как это неофициальное [[API]].
    

---