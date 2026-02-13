Вот **полное, подробное и максимально насыщенное** руководство по сравнению **`class`** и **`actor`** в Swift — актуально на 2026 год (Swift 6+ и строгий режим конкурентности).

### 1. Основное назначение и философия (2026)

| Характеристика                  | class                                      | actor                                      | Кто побеждает в 2026 году |
|---------------------------------|--------------------------------------------|--------------------------------------------|----------------------------|
| Основная цель                   | Объектно-ориентированное программирование | Потокобезопасное хранение изменяемого состояния | actor (для многопоточности) |
| Семантика                       | Reference type (по ссылке)                 | Reference type + изоляция состояния        | actor (безопаснее)         |
| Потокобезопасность              | Отсутствует по умолчанию                   | Полная, встроенная                         | actor                      |
| Когда использовать в новом коде | UI-объекты (UIViewController), legacy, протоколы без состояния | Всё изменяемое состояние в многопотоке     | actor в 90% случаев        |

**Короткий вердикт 2026 года**:
> Если объект **имеет изменяемое состояние** и будет использоваться из нескольких задач / потоков → **actor**.  
> Если это **UI-объект**, **singleton без состояния** или **старый код** → **class**.

### 2. Сравнение ключевых свойств (таблица 2026)

| Свойство                              | class                                      | actor                                      | Победитель и комментарий |
|---------------------------------------|--------------------------------------------|--------------------------------------------|---------------------------|
| Потокобезопасность                    | Нет (нужны lock / queue / @MainActor)      | Да (встроенная изоляция)                   | actor                     |
| Доступ к свойствам извне              | Прямой (`object.property`)                 | Только через `await` (`await actor.property`) | class удобнее, actor безопаснее |
| Доступ внутри типа                    | Прямой                                     | Прямой (без await)                         | Оба одинаково             |
| Наследование                          | Полное (class → class)                     | Запрещено (actor не наследуется)           | class                     |
| Реализация протоколов                 | Да                                         | Да                                         | Оба                       |
| Sendable по умолчанию                 | Нет (нужен @unchecked Sendable)            | Да (автоматически)                         | actor                     |
| Можно хранить в @MainActor            | Да                                         | Да (но избыточно)                          | Оба                       |
| Можно использовать в TaskGroup        | Да (если Sendable)                         | Да                                         | actor проще               |
| Размер экземпляра                     | Только указатель (8 байт)                  | Только указатель + executor                | class чуть легче          |
| Copy-on-Write (Array, Dictionary и т.д.) | Работает                                   | Работает                                   | Оба                       |
| Использование в SwiftUI               | Через @StateObject / @ObservedObject       | Через @StateObject + @MainActor            | @MainActor + class чаще   |

### 3. Самые важные различия в примерах (2026 стиль)

#### Пример 1 — Гонка данных (class vs actor)

```swift
// class → гонка данных
final class CounterClass {
    var value = 0
    func increment() { value += 1 }
}

let classCounter = CounterClass()

Task.detached { for _ in 0..<1000 { classCounter.increment() } }
Task.detached { for _ in 0..<1000 { classCounter.increment() } }

// value почти никогда не будет ровно 2000
```

```swift
// actor → всегда корректно
actor CounterActor {
    private var value = 0
    func increment() { value += 1 }
    var currentValue: Int { value }
}

let actorCounter = CounterActor()

Task.detached { for _ in 0..<1000 { await actorCounter.increment() } }
Task.detached { for _ in 0..<1000 { await actorCounter.increment() } }

print(await actorCounter.currentValue) // всегда 2000
```

#### Пример 2 — nonisolated (когда actor может быть "как class")

```swift
actor Session {
    private var token: String = ""
    
    // изолированный метод — требует await
    func setToken(_ new: String) {
        token = new
    }
    
    // nonisolated — доступен без await
    nonisolated func version() -> String {
        "2.1.0"
    }
    
    nonisolated var appName: String {
        "MyApp"
    }
}

let session = Session()

print(session.appName)          // без await
print(session.version())        // без await

Task {
    await session.setToken("xyz") // нужен await
}
```

#### Пример 3 — actor внутри @MainActor (очень частый паттерн)

```swift
actor DataService {
    private var users: [User] = []
    func add(_ user: User) { users.append(user) }
    func all() -> [User] { users }
}

@MainActor
class UserListViewModel: ObservableObject {
    @Published var users: [User] = []
    
    let service = DataService()
    
    func load() {
        Task {
            let loaded = await service.all()
            users = loaded  // безопасно — мы на @MainActor
        }
    }
}
```

### 4. Когда использовать class, а когда actor (рекомендации 2026)

| Ситуация                                      | Рекомендация 2026                  | Почему |
|-----------------------------------------------|-------------------------------------|--------|
| UI-классы (UIViewController, UIView)          | class + @MainActor                  | UIKit/AppKit требует class |
| ViewModel / ObservableObject                  | @MainActor class                    | SwiftUI / Combine ожидают @MainActor |
| Изменяемое состояние в многопотоке           | actor                               | Автоматическая безопасность |
| Immutable данные / конфиг                     | struct (Sendable)                   | Копирование, дешёвый захват |
| Legacy-код / Objective-C обёртки              | class + @unchecked Sendable         | Совместимость |
| Глобальный сервис (логи, аналитика, БД)       | @globalActor или обычный actor      | Централизация |
| Параллельные независимые задачи               | TaskGroup / async let               | Нет нужды в actor |

### 5. Лучшие практики 2026 года

- **actor для всего изменяемого состояния** в многопотоке  
- **@MainActor** — для всех ViewModel, контроллеров, UI-логики  
- **struct** — для immutable данных, моделей, DTO  
- **nonisolated** — используй для методов, которые не трогают состояние  
- **Не делай actor слишком большим** — 5–15 свойств и методов максимум  
- **Передавай actor по ссылке** — `let service = DataService()`  
- **Swift 6 strict concurrency** — включай полную проверку — она ловит почти все ошибки  
- **Тестирование** — создавай экземпляры actor и вызывай через `await`  
- **Мониторинг** — Instruments → Swift Tasks + Thread Sanitizer

**Короткий девиз 2026**:
> «actor — это class, который не даёт тебе сломать себе многопоточность.  
> В 2026 году почти всё изменяемое состояние должно жить в actor.  
> UI — в @MainActor class.  
> Immutable — в struct.  
> Legacy — в class с осторожностью.»

Удачи с безопасным, современным и читаемым многопоточным кодом в Swift! 🛡️