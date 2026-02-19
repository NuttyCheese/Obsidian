**`Result`** — это **стандартный enum** в [[Swift]] (добавлен в Swift 5), который предназначен для **явного возврата** либо успешного значения, либо ошибки.

```swift
enum Result<Success, Failure: Error> {
    case success(Success)
    case failure(Failure)
}
```

Это наиболее популярный и рекомендуемый способ возвращать результат операции в 2025–2026 годах, особенно в **синхронном** и **асинхронном** коде, когда нужно **явно** показать, что может быть либо успех, либо конкретная ошибка.

### Почему Result лучше, чем альтернативы (2026 взгляд)

| Альтернатива                 | Плюсы Result над ней                                    | Когда всё ещё используют альтернативу                 |
| ---------------------------- | ------------------------------------------------------- | ----------------------------------------------------- |
| `T?` ([[Optional]]) + throws | Один тип возврата → легче читать, меньше вложенности    | Когда ошибка не нужна (просто успех/не успех)         |
| `throws` напрямую            | Нет необходимости в do-catch на каждом вызове           | Когда ошибка должна быть обработана немедленно        |
| Tuple `(T?, Error?)`         | Типобезопасно: нельзя одновременно success и failure    | —                                                     |
| Custom enum                  | Стандартизировано, есть map/flatMap, совместимо с async | Когда нужны дополнительные кейсы (loading, cancelled) |

### Самые популярные паттерны Result в 2026 году

#### 1. Простой синхронный Result (самый частый)

```swift
enum ValidationError: Error {
    case empty
    case tooShort(min: Int)
}

func validateUsername(_ username: String) -> Result<String, ValidationError> {
    guard !username.isEmpty else {
        return .failure(.empty)
    }
    
    guard username.count >= 3 else {
        return .failure(.tooShort(min: 3))
    }
    
    return .success(username)
}

let result = validateUsername("al")
switch result {
case .success(let name):
    print("Валидное имя:", name)
case .failure(let error):
    print("Ошибка:", error)
}
```

#### 2. Обёртка throwing-функции в Result (очень популярно)

```swift
func fetchUser(id: UUID) throws -> User {
    // ... сетевой запрос или Core Data
}

func safeFetchUser(id: UUID) -> Result<User, Error> {
    Result { try fetchUser(id: id) }  // Swift 5.5+ синтаксис
}

// или классически
func safeFetchUser(id: UUID) -> Result<User, Error> {
    do {
        return .success(try fetchUser(id: id))
    } catch {
        return .failure(error)
    }
}
```

#### 3. Async + Result (самый частый паттерн в 2026)

```swift
func fetchPosts() async -> Result<[Post], Error> {
    do {
        let (data, _) = try await URLSession.shared.data(from: postsURL)
        let posts = try JSONDecoder().decode([Post].self, from: data)
        return .success(posts)
    } catch {
        return .failure(error)
    }
}

Task {
    let result = await fetchPosts()
    switch result {
    case .success(let posts):
        updateUI(with: posts)
    case .failure(let error):
        showError(error)
    }
}
```

#### 4. [[map]] / [[flatMap]] / recover (функциональный стиль)

```swift
let result: Result<Int, Error> = .success(10)

let doubled = result.map { $0 * 2 }              // success(20)
let string  = result.map { String($0) }          // success("10")

let recovered = result.recover { _ in 0 }        // success(10) или success(0) при ошибке

let flatMapped = result.flatMap { value in
    Result { value + 1 }                         // можно возвращать другой Result
}
```

### 5. Лучшие практики Result в Swift 2026

- **Используй `Result { try ... }`** (Swift 5.5+) — самый чистый способ оборачивать throwing-код  
- **Предпочитай `Result` над `throws`** в публичных API, если вызывающий код хочет обрабатывать ошибки позже  
- **Не возвращай `Result<Void, Error>`** — лучше `Result<Bool, Error>` или просто `throws`  
- **Для loading / cancelled / partial states** — создавай свой enum (не Result)  
- **В SwiftUI** — часто комбинируй с `@State` / `@Published` и `.task { await ... }`  
- **Swift 6 strict concurrency** — `Result` полностью `Sendable`, если `Success` и `Failure` — `Sendable`  
- **Документируйте** — пиши комментарий «Result — успех с данными или ошибка валидации»

**Короткий девиз 2026**:
> `Result` — это когда ты хочешь **явно** сказать: «либо успех с данными, либо конкретная ошибка».  
> В 2026 году:  
> - `Result { try ... }` — золотой стандарт обёртки throwing-функций  
> - `switch` / `if case` — для обработки  
> - `map` / `flatMap` / `recover` — для функционального стиля  
> - в async-коде → `await` + `Result` — основной паттерн  
> Это **основа** читаемого и безопасного кода с ошибками.
