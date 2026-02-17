**Vapor** — это **самый мощный и популярный серверный фреймворк** на [[Swift]] для создания:

- [[Swift/REST API]]  
- Web-приложений  
- микросервисов  
- [[WebSocket]]-серверов  
- [[GraphQL]]-серверов  
- серверов для IoT и ботов

Vapor 4+ (на 2026 год — версия 4.100+) использует **Swift Concurrency** ([[async]]/[[await]]), **NIO** (non-blocking I/O) и **Fluent** (ORM).

### 1. Почему Vapor в 2026 году всё ещё №1

| Преимущество                                       | Почему это важно в 2026               |
| -------------------------------------------------- | ------------------------------------- |
| Полностью нативный Swift                           | Типобезопасность, производительность  |
| Async/await из коробки                             | Современный код без [[callback]] hell |
| Fluent ORM + [[PostgreSQL]], [[SQLite]], [[MySQL]] | Мощный и гибкий доступ к БД           |
| Встроенный JWT, OAuth, Redis, WebSocket            | Всё для реальных приложений           |
| Отличная документация и комьюнити                  | 10 000+ звёзд на [[GitHub]]           |
| Docker + Vapor Toolbox                             | Легко деплоить в Kubernetes / Fly.io  |
| Высокая производительность (NIO)                   | 100 000+ req/s на одном ядре          |

### 2. Установка и первый проект (2026)

#### Установка Vapor Toolbox

```bash
# macOS (Homebrew)
brew install vapor

# Linux (Ubuntu / Debian)
sudo apt-get install vapor

# Проверить версию
vapor --version
# Должно быть 19.x.x или выше (2026)
```

#### Создание нового проекта

```bash
vapor new MyBackend --branch main
cd MyBackend
vapor xcode
```

Выбери:

- Xcode project — Yes  
- Open Xcode project — Yes  

### 3. Структура проекта Vapor (стандарт 2026)

```
MyBackend/
├── Sources/
│   ├── App/
│   │   ├── configure.swift       # Настройка роутов, БД, middleware
│   │   ├── routes.swift          # Все маршруты
│   │   ├── Models/               # Fluent-модели
│   │   ├── Controllers/          # Контроллеры
│   │   ├── Migrations/           # Миграции БД
│   │   └── Middleware/           # Middleware
│   └── Run/                      # Точка входа (main.swift)
├── Tests/                        # Тесты
├── Public/                       # Статические файлы (CSS, JS, изображения)
├── Resources/                    # Views (Leaf), seeds
├── Package.swift                 # Зависимости
└── docker-compose.yml            # PostgreSQL, Redis (опционально)
```

### 4. Базовые примеры кода (от простого к сложному)

#### Пример 1 — Hello World + маршруты

```swift
// routes.swift
import Vapor

func routes(_ app: Application) throws {
    // GET /
    app.get { req in
        "Hello, Vapor 4!"
    }
    
    // GET /hello/:name
    app.get("hello", ":name") { req -> String in
        let name = req.parameters.get("name") ?? "гость"
        return "Привет, \(name)!"
    }
    
    // GET /json
    app.get("json") { req -> [String: String] in
        ["message": "Hello JSON", "status": "ok"]
    }
}
```

#### Пример 2 — [[JSON]]-модель + [[Codable]]

```swift
// Models/User.swift
import Vapor

struct User: Content {
    let id: UUID
    let name: String
    let email: String
    let age: Int?
}

// routes.swift
app.get("users") { req -> [User] in
    [
        User(id: UUID(), name: "Анна", email: "anna@example.com", age: 28),
        User(id: UUID(), name: "Боб", email: "bob@example.com", age: 35)
    ]
}

app.post("users") { req -> User in
    let user = try req.content.decode(User.self)
    // сохранение в БД (см. Fluent ниже)
    return user
}
```

#### Пример 3 — Fluent + PostgreSQL (самый частый стек 2026)

```swift
// Package.swift — добавляем зависимости
.package(url: "https://github.com/vapor/fluent-postgres-driver.git", from: "2.9.0")

// configure.swift
import Fluent
import FluentPostgresDriver

public func configure(_ app: Application) async throws {
    app.databases.use(.postgres(
        hostname: Environment.get("DATABASE_HOST") ?? "localhost",
        port: Environment.get("DATABASE_PORT").flatMap(Int.init) ?? 5432,
        username: Environment.get("DATABASE_USERNAME") ?? "vapor",
        password: Environment.get("DATABASE_PASSWORD") ?? "password",
        database: Environment.get("DATABASE_NAME") ?? "vapor"
    ), as: .psql)
    
    app.migrations.add(CreateUser())
    
    // Регистрация миграций
    try await app.autoMigrate()
}

// Models/User.swift
import Fluent
import Vapor

final class User: Model, Content {
    static let schema = "users"
    
    @ID(key: .id)
    var id: UUID?
    
    @Field(key: "name")
    var name: String
    
    @Field(key: "email")
    var email: String
    
    @Field(key: "age")
    var age: Int?
    
    init() { }
    
    init(id: UUID? = nil, name: String, email: String, age: Int? = nil) {
        self.id = id
        self.name = name
        self.email = email
        self.age = age
    }
}

// Migrations/CreateUser.swift
import Fluent

struct CreateUser: AsyncMigration {
    func prepare(on database: Database) async throws {
        try await database.schema("users")
            .id()
            .field("name", .string, .required)
            .field("email", .string, .required)
            .field("age", .int)
            .create()
    }
    
    func revert(on database: Database) async throws {
        try await database.schema("users").delete()
    }
}
```

#### Пример 4 — JWT-аутентификация (очень популярно в 2026)

```swift
// Package.swift — добавляем vapor/jwt
.package(url: "https://github.com/vapor/jwt.git", from: "4.2.0")

// configure.swift
import JWT

app.jwt.signers.use(.hs256(key: "secret-key"))

// Модель токена
struct UserPayload: JWTPayload, Authenticatable {
    var userID: UUID
    var exp: ExpirationClaim
    
    func verify(using signer: JWTSigner) throws {
        try exp.verifyNotExpired()
    }
}

// Middleware
let protected = app.grouped(UserPayload.authenticator(), UserPayload.guardMiddleware())
protected.get("me") { req -> UserPayload in
    try req.auth.require(UserPayload.self)
}
```

#### Пример 5 — WebSocket-чат

```swift
app.webSocket("chat") { req, ws in
    ws.onText { ws, text in
        // Рассылка всем подключённым
        for client in req.application.webSocketClients.active {
            client.send(text)
        }
    }
    
    ws.onClose.whenComplete { _ in
        req.application.webSocketClients.remove(ws)
    }
    
    req.application.webSocketClients.add(ws)
}
```

### 5. Таблица: ключевые компоненты Vapor

| Компонент              | Назначение                                   | Пример использования |
|------------------------|----------------------------------------------|----------------------|
| `Application`          | Главный объект приложения                    | `let app = Application()` |
| `routes.swift`         | Регистрация всех маршрутов                   | `app.get("hello") { ... }` |
| `configure.swift`      | Настройка БД, JWT, middleware, порта         | `app.databases.use(...)` |
| `Fluent`               | ORM для PostgreSQL, SQLite, MySQL            | `Model`, `Migration` |
| `Middleware`           | Перехват запросов/ответов                    | Логирование, CORS, Auth |
| `Content`              | Протокол для Codable-моделей в запросах      | `struct User: Content` |
| `WebSocket`            | Реал-тайм соединения                         | `app.webSocket("chat")` |
| `JWT`                  | Аутентификация через токены                  | `app.jwt.signers.use(...)` |

### 6. Лучшие практики Vapor 2026

- Всегда используй **async/await** и **Fluent 4+**
- Модели — **Content** + **Model** + **Codable**
- Аутентификация — **JWT** или **OAuth2**
- Middleware — для логирования, CORS, rate limiting
- Миграции — всегда в отдельной папке
- Тесты — используй `XCTVapor` (встроенные тесты Vapor)
- Деплой — Docker + Fly.io, Render, Railway, Kubernetes
- Мониторинг — Prometheus + Grafana
- Логи — `app.logger` + Sentry / Logflare

**Короткий девиз**:
> «Vapor 2026 — это Swift на сервере без компромиссов: типобезопасность, производительность, async/await, Fluent, JWT и WebSocket из коробки.  
> Пиши бэкенд так же красиво, как фронтенд.»
