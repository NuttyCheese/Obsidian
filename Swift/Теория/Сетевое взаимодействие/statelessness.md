#network #Swift 
Вот **полная, подробная и максимально насыщенная** статья про **Statelessness** (безсостоятельность) в [[RESTful API]] с множеством примеров кода, сравнительных таблиц, реальных сценариев и рекомендаций для [[Swift]]/[[iOS]]-разработки (актуально на 2026 год).

# Statelessness в RESTful API — полное руководство

**Statelessness** (безсостоятельность) — один из **шести фундаментальных принципов REST**, сформулированных Роем Филдингом в 2000 году.

**Определение**:  
Каждый запрос от клиента к серверу должен содержать **всю необходимую информацию** для его обработки.  
Сервер **не хранит** состояние клиента между запросами.  
Каждый запрос **самодостаточен** и **независим** от предыдущих.

### Почему statelessness — это важно

| Преимущество                          | Почему это критично в 2026 году                                                                 |
|---------------------------------------|-------------------------------------------------------------------------------------------------|
| Масштабируемость                      | Любой сервер может обработать любой запрос — нет привязки к сессии                             |
| Отказоустойчивость                    | Сервер упал → следующий запрос обрабатывается другим сервером без потери состояния              |
| Простота архитектуры                  | Нет необходимости синхронизировать состояние между серверами (Redis, sticky sessions и т.д.)   |
| Параллелизм и кэширование             | Запросы можно кэшировать, распараллеливать, балансировать без сложной логики                    |
| Простота отладки                      | Каждый запрос — независимый, легко воспроизвести и протестировать                              |

### Stateful vs Stateless — сравнение на примерах

#### Пример 1 — Stateful [[API]] (нарушение принципа)

```http
# Запрос 1 — авторизация
POST /login
{
  "username": "john",
  "password": "secret123"
}
→ Ответ: 200 OK + Set-Cookie: session=abc123

# Запрос 2 — получение профиля (сессия на сервере)
GET /profile
Cookie: session=abc123
→ Сервер смотрит в Redis/memcached: session=abc123 → userId=456 → возвращает профиль
```

**Проблемы**:
- Сервер хранит сессию → привязка к конкретному серверу
- Масштабирование требует Redis/Memcached или sticky sessions
- Если сессия потерялась — пользователь разлогинен
- Трудно тестировать и отлаживать

#### Пример 2 — Stateless API (правильный RESTful подход)

```http
# Запрос 1 — авторизация
POST /login
{
  "username": "john",
  "password": "secret123"
}
→ Ответ: 200 OK
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}

# Запрос 2 — получение профиля (всё в запросе)
GET /profile
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
→ Сервер проверяет JWT → достаёт userId → возвращает профиль
```

**Преимущества**:
- Нет хранения состояния на сервере
- Любой сервер может обработать запрос
- Масштабируется горизонтально без проблем
- Легко тестировать (curl/postman)

### Как statelessness реализуется в Swift (клиент + сервер)

#### Клиентская сторона (iOS-приложение)

```swift
// Пример: авторизация + получение профиля (stateless)

struct LoginRequest: Encodable {
    let username: String
    let password: String
}

struct LoginResponse: Decodable {
    let token: String
}

struct UserProfile: Decodable {
    let id: Int
    let name: String
}

class AuthService {
    private var token: String?
    
    func login(username: String, password: String) async throws -> String {
        let url = URL(string: "https://api.example.com/login")!
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.httpBody = try JSONEncoder().encode(LoginRequest(username: username, password: password))
        
        let (data, _) = try await URLSession.shared.data(for: request)
        let response = try JSONDecoder().decode(LoginResponse.self, from: data)
        token = response.token
        return response.token
    }
    
    func fetchProfile() async throws -> UserProfile {
        guard let token = token else { throw AuthError.notAuthenticated }
        
        var request = URLRequest(url: URL(string: "https://api.example.com/profile")!)
        request.httpMethod = "GET"
        request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        
        let (data, _) = try await URLSession.shared.data(for: request)
        return try JSONDecoder().decode(UserProfile.self, from: data)
    }
}
```

**Ключ**:  
Весь контекст (аутентификация) передаётся в каждом запросе через заголовок `Authorization`.  
Сервер **не хранит** сессию.

#### Серверная сторона ([[Vapor]] — популярный фреймворк в 2026)

```swift
import Vapor

struct UserController: RouteCollection {
    func boot(routes: RoutesBuilder) throws {
        let users = routes.grouped("api", "users")
        users.get(use: getAll)
        users.get(":id", use: getById)
        users.post(use: create)
        users.patch(":id", use: update)
        users.delete(":id", use: delete)
    }
    
    func getAll(req: Request) async throws -> [User.Public] {
        // Каждый запрос независим — берём из БД напрямую
        try await User.query(on: req.db).all().map { $0.public }
    }
    
    func getById(req: Request) async throws -> User.Public {
        guard let id = req.parameters.get("id", as: UUID.self) else {
            throw Abort(.badRequest)
        }
        guard let user = try await User.find(id, on: req.db) else {
            throw Abort(.notFound)
        }
        return user.public
    }
    
    // ... остальные методы аналогично
}
```

**Ключ**:  
Нет хранения сессии на сервере.  
Каждый запрос содержит токен или другой идентификатор → сервер проверяет его и выполняет действие.

### Таблица сравнения stateful vs stateless

| Характеристика                  | Stateful API                               | Stateless (RESTful) API                     |
|---------------------------------|--------------------------------------------|---------------------------------------------|
| Хранение состояния              | На сервере (сессия, Redis)                 | В запросе (токен, JWT)                      |
| Масштабирование                 | Сложно (sticky sessions, синхронизация)    | Просто (любой сервер)                       |
| Отказоустойчивость              | Средняя                                    | Высокая                                     |
| Тестирование                    | Сложнее (нужна сессия)                     | Легко (каждый запрос независим)            |
| Кэширование                     | Ограничено                                 | Полное (ETag, Cache-Control)                |
| Сложность клиента               | Простая (cookie)                           | Нужно передавать токен в каждом запросе     |

### Лучшие практики stateless REST в Swift 2026

- **Аутентификация** — JWT (Bearer Token) в заголовке `Authorization`
- **Состояние клиента** — передавай в каждом запросе (query, header, body)
- **Идемпотентность** — GET, PUT, DELETE должны быть идемпотентными
- **Кэширование** — используй `ETag`, `Last-Modified`, `Cache-Control`
- **HATEOAS** (опционально) — возвращай ссылки в ответах (редко в мобильных API)
- **Версионирование** — `/api/v1/users` или заголовок `Accept: application/vnd.example.v1+json`

**Короткое правило 2026**:
> «RESTful = stateless + правильные HTTP-методы + ресурсы в URI.  
> В Swift 2026 — каждый запрос должен быть **самодостаточным** (токен, параметры, тело).  
> Нет сессий на сервере — только проверка токена и выполнение действия.»
