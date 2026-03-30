**`func`** — ключевое слово в [[Swift]] для объявления **функции** (или метода).  
Функция — это именованный блок кода, который выполняет задачу, может принимать параметры, возвращать значение и выбрасывать ошибки.

> Проще говоря: `func` = «даёшь имя куску кода, чтобы потом его можно было вызвать много раз».

### 1. Основные формы функций в Swift (2026 актуально)

| Тип функции                       | Синтаксис (кратко)                                 | Когда использовать (самые частые кейсы 2026)            |
| --------------------------------- | -------------------------------------------------- | ------------------------------------------------------- |
| Простая без параметров и возврата | `func sayHello() { ... }`                          | Логирование, сайд-эффекты, утилиты                      |
| С параметрами и возвратом         | `func add(a: Int, b: Int) -> Int { ... }`          | Математика, преобразования, бизнес-логика               |
| С [[inout]]-параметром            | `func increment(value: inout Int) { ... }`         | Изменение внешней переменной                            |
| [[Throws]]-функция                | `func fetch() throws -> Data { ... }`              | Работа с сетью, файлами, парсингом                      |
| [[Async]]-функция                 | `func load() async throws -> User { ... }`         | Асинхронные операции (сеть, диск, [[API]])              |
| [[Generic]]-функция               | `func swap<T>(_ a: inout T, _ b: inout T) { ... }` | Универсальные алгоритмы ([[swapAt]], [[sort]], [[map]]) |
| Метод в типе                      | `func greet() { ... }` внутри struct/class/enum    | Поведение объекта                                       |
| [[Closure]] как параметр          | `func asyncTask(completion: @escaping () -> Void)` | Колбэки, completion handlers                            |

### 2. Полный синтаксис и все ключевые возможности (примеры)

#### 2.1. Базовая функция

```swift
func greet(name: String) -> String {
    return "Привет, \(name)!"
}

let message = greet(name: "Алексей")
print(message) // Привет, Алексей!
```

#### 2.2. Функция без возврата (Void)

```swift
func logEvent(_ event: String) {
    print("Событие: \(event)")
    // или Analytics.track(event)
}
```

#### 2.3. Функция с несколькими параметрами и метками

```swift
func move(from start: CGPoint, to end: CGPoint, duration: TimeInterval) {
    // анимация
}

move(from: CGPoint(x: 0, y: 0), to: CGPoint(x: 100, y: 200), duration: 0.3)
```

#### 2.4. Inout-параметры (изменение внешней переменной)

```swift
func increment(by value: Int, _ number: inout Int) {
    number += value
}

var count = 10
increment(by: 5, &count)
print(count) // 15
```

#### 2.5. Throws + [[do-catch]] (самый частый паттерн 2026)

```swift
enum NetworkError: Error {
    case offline
    case timeout(statusCode: Int)
}

func fetchUser(id: String) throws -> User {
    if Bool.random() { throw NetworkError.offline }
    // ...
    return User(id: id)
}

do {
    let user = try fetchUser(id: "123")
    print("Пользователь:", user.name)
} catch NetworkError.offline {
    print("Нет интернета")
} catch let error as NetworkError where error == .timeout(statusCode: 504) {
    print("Таймаут сервера")
} catch {
    print("Неизвестная ошибка:", error.localizedDescription)
}
```

#### 2.6. Async + throws (современный стандарт)

```swift
func fetchUserAsync(id: String) async throws -> User {
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(User.self, from: data)
}

Task {
    do {
        let user = try await fetchUserAsync(id: "123")
        await MainActor.run {
            updateUI(with: user)
        }
    } catch {
        await showError(error.localizedDescription)
    }
}
```

#### 2.7. Generic-функция (очень мощно)

```swift
func first<T: Equatable>(_ items: [T], matching target: T) -> T? {
    items.first { $0 == target }
}

let numbers = [1, 2, 3, 4, 5]
if let found = first(numbers, matching: 3) {
    print("Нашли:", found)
}
```

### 3. Лучшие практики func в Swift 2026

- **Назови функцию так**, чтобы её вызов читался как предложение:
  - `fetchUser(id:)` → хорошо
  - `getUserById(id:)` → хуже
  - `user(withId:)` → ещё лучше (SwiftUI-стиль)

- **Используй метки параметров** — улучшают читаемость вызова

- **Делай функции короткими** — идеально 5–15 строк, максимум 30

- **Одна ответственность** — функция должна делать **одну** вещь

- **Throws** — только если ошибка реально возможна и должна быть обработана

- **Async** — используй `async throws` для всех I/O-операций (сеть, диск, база данных)

- **Generics** — применяй, когда функция может работать с разными типами

- **Inout** — используй осторожно, лучше возвращать новое значение

- **Swift 6 strict concurrency** — помечай функции [[@MainActor]], если обновляют UI, или делай их [[nonisolated]]

- Документируй — пиши комментарий «Возвращает пользователя по ID или выбрасывает AuthError»

**Короткий девиз 2026**:
> `func` — это именованный кусок логики, который можно вызвать снова и снова.  
> В 2026 году:  
> - делай короткие, читаемые функции с понятными именами  
> - используй `async throws` для асинхронных операций  
> - `throws` — только если ошибка реально нужна наверху  
> - generics и inout — мощные, но используй с умом  
> Это **основа** чистого и надёжного кода в Swift.
