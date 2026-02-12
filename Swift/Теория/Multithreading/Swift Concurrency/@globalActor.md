## 1. Что это такое?

`@globalActor` — это **атрибут**, которым мы помечаем специальный `actor`, объявленный как **глобальный диспетчер**.

- Глобальный [[actor]] — это **единственный экземпляр actor**, доступный всему приложению.
    
- Когда вы помечаете метод, свойство или класс аннотацией `@MyGlobalActor`, [[Swift]] гарантирует, что **весь код будет выполняться в его изоляции**.
    
- Это позволяет централизовать синхронизацию и избежать гонок данных.
    

👉 Самый известный пример — [[@MainActor]]. Он — глобальный actor, встроенный в Swift, который гарантирует выполнение кода на **главном потоке (UI-thread)**.

---

## 2. Как это работает?

1. Создаётся `actor` с атрибутом `@globalActor`.
    
2. Внутри него должен быть `static let shared = <имя>` — это обязательное условие, чтобы был один экземпляр.
    
3. Всё, что помечено `@YourActor`, автоматически будет выполняться в этом actor.
    

---

## 3. Объявление глобального actor

```swift
import Foundation

// 1. Создаём actor
actor DatabaseActor { }

// 2. Делаем его глобальным
@globalActor
struct Database {
    static let shared = DatabaseActor()
}
```

Теперь `Database` — это глобальный actor.

---

## 4. Использование в коде

### Пример 1. Пометка метода

```swift
@Database
func saveToDB(_ data: String) {
    print("Сохраняем:", data)
}
```

👉 этот метод всегда будет выполняться внутри `DatabaseActor`.

---

### Пример 2. Пометка класса

```swift
@Database
class DataManager {
    private var storage: [String] = []

    func add(_ item: String) {
        storage.append(item)
    }

    func allItems() -> [String] {
        return storage
    }
}
```

Теперь **все методы и свойства этого класса выполняются в `DatabaseActor`**.

---

### Пример 3. Вызовы с `await`

```swift
let manager = DataManager()

Task {
    await manager.add("яблоко")
    await manager.add("банан")
    print(await manager.allItems()) // ["яблоко", "банан"]
}
```

---

### Пример 4. Смешивание с `@MainActor`

```swift
@Database
class DatabaseService {
    func fetch() -> [String] {
        return ["данные"]
    }
}

@MainActor
class ViewModel: ObservableObject {
    @Published var items: [String] = []

    let service = DatabaseService()

    func load() {
        Task {
            let data = await service.fetch()
            items = data // безопасно, так как ViewModel работает на MainActor
        }
    }
}
```

---

### Пример 5. Несколько глобальных actors

```swift
actor FileActor { }
actor NetworkActor { }

@globalActor
struct FileManagerActor {
    static let shared = FileActor()
}

@globalActor
struct NetworkManagerActor {
    static let shared = NetworkActor()
}

@FileManagerActor
func readFile() -> String {
    return "Содержимое файла"
}

@NetworkManagerActor
func fetchData() -> String {
    return "Данные из сети"
}
```

---

### Пример 6. Сравнение с обычным actor

```swift
actor SimpleStorage {
    private var items: [String] = []
    
    func add(_ item: String) {
        items.append(item)
    }
}

let storage = SimpleStorage()

Task {
    await storage.add("test") // экземплярный actor
}

@Database
func addGlobal(_ item: String) {
    print("Добавлено:", item)
}

Task {
    await addGlobal("test") // глобальный actor
}
```

---

## 5. Зачем нужен `@globalActor`?

- Чтобы создать **свою глобальную очередь** для задач, которые должны выполняться в одном потоке (например, работа с БД, логирование, синхронизация).
    
- Чтобы централизованно управлять доступом к общим данным.
    
- Чтобы **явно указать**, где должен выполняться код (как `@MainActor` для UI).
    

---

## 6. Итоги

- `@globalActor` создаёт **глобальный изолированный контекст** выполнения.
    
- Он работает как `@MainActor`, но вы можете создавать **свои собственные**.
    
- Всё, что помечено этим атрибутом, будет выполняться внутри одного глобального actor.
    

---
