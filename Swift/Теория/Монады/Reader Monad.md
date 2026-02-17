**Reader Monad** — это монада, которая позволяет **передавать одно и то же окружение** (environment / config / dependencies) через цепочку функций **без изменения их сигнатуры** и без необходимости передавать этот контекст вручную в каждую функцию.

Представьте, что у вас есть:

- Глобальная конфигурация ([[API]] [[URL]], ключи, база данных, логгер, текущий пользователь и т.д.)
- Много функций, которые используют эту конфигурацию

**Без Reader**:
```swift
func fetchUser(config: Config, token: String) -> User { ... }
func processUser(config: Config, user: User) -> Result { ... }
func saveUser(config: Config, result: Result) -> Void { ... }
```

**С Reader**:
```swift
func fetchUser() -> Reader<Config, User> { ... }
func processUser(user: User) -> Reader<Config, Result> { ... }
func saveUser(result: Result) -> Reader<Config, Void> { ... }

// Всё комбинируется без явной передачи config
let program = fetchUser()
    .flatMap(processUser)
    .flatMap(saveUser)

// Запуск: program.run(myConfig)
```

Reader Monad **явно** показывает в типе, что функция зависит от окружения, но **скрывает** передачу этого окружения.

### Основные операции Reader Monad

| Операция        | Оператор / Метод  | Что делает                                                                | Пример                                 |
| --------------- | ----------------- | ------------------------------------------------------------------------- | -------------------------------------- |
| **unit / pure** | `Reader.pure(_:)` | Упаковывает обычное значение в Reader                                     | `Reader.pure(42)`                      |
| **[[map]]**     | `map`             | Преобразует результат, не меняя окружение                                 | `reader.map { $0 * 2 }`                |
| **[[flatMap]]** | `flatMap`         | Композиция: берёт результат → вызывает функцию, возвращающую новый Reader | `reader.flatMap { ... }`               |
| **run**         | `run(_ env: Env)` | Запускает вычисление с конкретным окружением                              | `program.run(config)`                  |
| **ask**         | `ask()`           | Получает текущее окружение внутри Reader                                  | `Reader.ask()`                         |
| **local**       | `local`           | Изменяет окружение только для вложенного Reader                           | `reader.local { $0.withToken("new") }` |

### Полная реализация Reader Monad в [[Swift]] (2026)

```swift
struct Reader<Environment, Value> {
    let run: (Environment) -> Value
    
    init(_ run: @escaping (Environment) -> Value) {
        self.run = run
    }
    
    // unit / pure
    static func pure(_ value: Value) -> Reader<Environment, Value> {
        Reader { _ in value }
    }
    
    // map — Functor
    func map<B>(_ transform: @escaping (Value) -> B) -> Reader<Environment, B> {
        Reader { env in
            transform(self.run(env))
        }
    }
    
    // flatMap — Monad
    func flatMap<B>(_ transform: @escaping (Value) -> Reader<Environment, B>) -> Reader<Environment, B> {
        Reader { env in
            transform(self.run(env)).run(env)
        }
    }
    
    // ask — получить текущее окружение
    static func ask() -> Reader<Environment, Environment> {
        Reader { env in env }
    }
    
    // local — изменить окружение для вложенного вычисления
    func local(_ modify: @escaping (Environment) -> Environment) -> Reader<Environment, Value> {
        Reader { env in
            self.run(modify(env))
        }
    }
}
```

### Реальные примеры использования

#### Пример 1 — Классический: конфигурация [[API]]

```swift
struct APIConfig {
    let baseURL: URL
    let apiKey: String
}

func fetchUser(id: Int) -> Reader<APIConfig, String> {
    Reader { config in
        "GET \(config.baseURL)/users/\(id) with key \(config.apiKey)"
    }
}

func logRequest(_ request: String) -> Reader<APIConfig, Void> {
    Reader { _ in
        print("Request: \(request)")
    }
}

let program = fetchUser(id: 42)
    .flatMap { request in
        logRequest(request).map { _ in request }
    }

let config = APIConfig(baseURL: URL(string: "https://api.example.com")!, apiKey: "xyz123")
let result = program.run(config)
// Вывод: Request: GET https://api.example.com/users/42 with key xyz123
```

#### Пример 2 — Reader + [[async]]/[[await]] (2026 стандарт)

```swift
struct AsyncReader<Environment, Value> {
    let run: (Environment) async -> Value
    
    func flatMap<B>(_ transform: @escaping (Value) async -> AsyncReader<Environment, B>) async -> AsyncReader<Environment, B> {
        AsyncReader { env in
            let value = await self.run(env)
            return await transform(value).run(env)
        }
    }
}

func fetchUserAsync(id: Int) -> AsyncReader<APIConfig, User> {
    AsyncReader { config in
        // имитация сетевого запроса
        try? await Task.sleep(nanoseconds: 1_000_000_000)
        return User(id: id, name: "User \(id)")
    }
}
```

#### Пример 3 — Dependency Injection через Reader

```swift
protocol Logger {
    func log(_ message: String)
}

struct ConsoleLogger: Logger {
    func log(_ message: String) { print("[LOG] \(message)") }
}

struct AppDependencies {
    let logger: Logger
    let apiConfig: APIConfig
}

func logStart() -> Reader<AppDependencies, Void> {
    Reader { deps in
        deps.logger.log("Приложение запущено")
    }
}

let program = logStart()
    .flatMap { _ in
        // другие шаги
        Reader<AppDependencies, Void>.pure(())
    }

let deps = AppDependencies(logger: ConsoleLogger(), apiConfig: APIConfig(...))
program.run(deps)
```

### 5. Сравнение Reader Monad с другими подходами

| Подход                     | Передача зависимостей | Чистота функций | Тестируемость | Композиция | Overhead |
|----------------------------|------------------------|------------------|---------------|------------|----------|
| Явная передача параметров  | Вручную в каждую функцию | Средняя          | Средняя       | Плохо      | Нет      |
| Reader Monad               | Через Reader<Env, A>   | Высокая          | Высокая       | Отлично    | Минимальный |
| Environment / Dependency Injection (TCA, swift-dependencies) | Через @Dependency | Высокая          | Отличная      | Отлично    | Минимальный |
| Global / Singleton         | Глобальные переменные  | Низкая           | Плохая        | Плохо      | Нет      |

### 6. Лучшие практики 2026

- Используйте **AsyncReader** с `async throws` для современных приложений  
- **Не выполняйте** `run()` внутри чистых функций — только в оболочке (App, [[SceneDelegate]], ViewModel)  
- Для тестов — создавайте **Mock Environment** и передавайте его в `run()`  
- Комбинируйте с **ReaderT** (Reader + другая монада) для сложных эффектов  
- Для больших приложений — рассмотрите **swift-dependencies** или **The Composable Architecture (TCA)** вместо ручного Reader  
- В Swift 6 — Reader Monad идеально вписывается в strict concurrency (явно показывает зависимости)

**Короткий девиз 2026**:
> «Reader Monad — это способ сказать: «Эта функция зависит от окружения, но я не хочу таскать его вручную».  
> Она делает код чистым, тестируемым и композируемым.  
> Запускай run() только в одном месте — и весь мир будет знать, где происходят побочные эффекты.»
