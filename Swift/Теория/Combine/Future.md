**`Future`** — это специальный **[[Publisher]]** в [[Combine]], который представляет **одноразовую асинхронную операцию**, которая завершится **ровно один раз**: либо успехом (`Output`), либо ошибкой (`Failure`).

После завершения (success или failure) он больше **никогда ничего не испустит** и автоматически завершается.

Это самый простой и естественный способ **интегрировать асинхронный код** (замыкания, [[completion handler]], [[DispatchQueue]], [[async]]/[[await]]) в реактивный поток Combine.

### Когда использовать Future (реальные сценарии 2025–2026)

| Сценарий                                       | Почему именно Future                                                        | Альтернатива (когда не Future)                    |
| ---------------------------------------------- | --------------------------------------------------------------------------- | ------------------------------------------------- |
| Обёртка над completion-handler API             | Старые [[API]] ([[URLSession]], [[Core Location]], [[AVFoundation]] и т.д.) | `Future` — классика                               |
| Преобразование async/[[await]] → [[Publisher]] | Вызов [[async]] функции внутри Combine-пайплайна                            | `async let` + `Task` (если не нужен поток)        |
| Одноразовый сетевой запрос без повторений      | Загрузка конфига, авторизация, загрузка профиля                             | `URLSession.dataTaskPublisher` (если нужен retry) |
| Тесты и мокинг асинхронных операций            | Легко создать Future с заранее известным результатом                        | `Just` / `Fail` / `Deferred`                      |
| Отложенное выполнение с задержкой              | Таймауты, симуляция сетевой задержки                                        | `Just(value).delay(for: ...)`                     |
| Обработка ошибки в цепочке Combine             | `.catch`, `.replaceError`, `.retry` после Future                            | —                                                 |

### Самые популярные и рекомендуемые паттерны (2026)

#### 1. Простейший Future (самый частый)

```swift
let future = Future<String, Never> { promise in
    promise(.success("Успех!"))
    // promise(.failure(MyError.failed)) — если ошибка
}

future
    .sink { value in
        print("Получено:", value)
    }
    .store(in: &cancellables)
```

#### 2. Future с реальной задержкой ([[DispatchQueue]])

```swift
func delayedGreeting() -> Future<String, Never> {
    Future { promise in
        DispatchQueue.global().asyncAfter(deadline: .now() + 2) {
            promise(.success("Привет через 2 секунды!"))
        }
    }
}

delayedGreeting()
    .sink { print($0) }
    .store(in: &cancellables)
```

#### 3. Future с возможной ошибкой (реальный сетевой пример)

```swift
enum NetworkError: Error {
    case invalidURL
    case noData
}

func fetchString(from urlString: String) -> Future<String, NetworkError> {
    Future { promise in
        guard let url = URL(string: urlString) else {
            promise(.failure(.invalidURL))
            return
        }
        
        URLSession.shared.dataTask(with: url) { data, _, error in
            if let error {
                promise(.failure(.noData))
                return
            }
            
            guard let data, let string = String(data: data, encoding: .utf8) else {
                promise(.failure(.noData))
                return
            }
            
            promise(.success(string))
        }.resume()
    }
}

fetchString(from: "https://example.com")
    .sink(receiveCompletion: { completion in
        if case .failure(let error) = completion {
            print("Ошибка:", error)
        }
    }, receiveValue: { text in
        print("Текст:", text)
    })
    .store(in: &cancellables)
```

#### 4. Самый современный способ: Future + async/await (iOS 15+)

```swift
func fetchUser(id: UUID) async throws -> User {
    // async код
    try await someAsyncAPI.fetchUser(id: id)
}

func fetchUserPublisher(id: UUID) -> Future<User, Error> {
    Future { promise in
        Task {
            do {
                let user = try await fetchUser(id: id)
                promise(.success(user))
            } catch {
                promise(.failure(error))
            }
        }
    }
}
```

#### 5. Future внутри цепочки Combine (очень популярно)

```swift
$username
    .debounce(for: .seconds(0.8), scheduler: RunLoop.main)
    .removeDuplicates()
    .flatMap { [weak self] username in
        self?.checkUsernameAvailability(username) ?? Fail(error: URLError(.badURL)).eraseToAnyPublisher()
    }
    .map { $0.isAvailable }
    .assign(to: &$isUsernameAvailable)
```

### Лучшие практики Future в Combine 2026

- **Всегда** возвращайте `Future` с конкретной `Failure` (например `Error`, `NetworkError`), а не `Never`, если может быть ошибка  
- **Используйте** `Future` только для **одноразовых операций** — для повторяющихся событий лучше `PassthroughSubject` или `CurrentValueSubject`  
- **Комбинируйте** с `.catch`, `.replaceError`, `.retry` — это делает цепочку устойчивой  
- **Для сетевых запросов** — чаще используйте `URLSession.dataTaskPublisher` (он уже Publisher), а Future — для обёртки над completion-handler API  
- **В SwiftUI** — Future обычно не нужен напрямую — используйте `.task` и `async let`  
- **Для тестов** — создавайте `Future` с `Just` / `Fail` / `Deferred`  
- **Документируйте** — пишите комментарий:

```swift
/// Асинхронная проверка доступности имени пользователя
func checkUsernameAvailability(_ username: String) -> Future<UsernameCheckResult, NetworkError>
```

**Короткий итог 2026**:
> `Future` — это **одноразовый Publisher**, который завершается ровно один раз (успех или ошибка).  
> В 2026 году:  
> - идеален для обёртки completion-handler API и async/await функций  
> - создаётся через `Future { promise in ... }`  
> - комбинируется с `flatMap`, `map`, `catch`, `retry`  
> - для повторяющихся событий используй другие Publisher (`PassthroughSubject`, `@Published`)  
> - это **самый простой** способ ввести асинхронный код в Combine-пайплайн  
