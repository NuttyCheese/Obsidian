**REST** — это **архитектурный стиль** для построения распределённых систем, предложенный **Роем Филдингом** в 2000 году в его диссертации.  
REST определяет принципы, по которым строятся **[[RESTful API]]** — самые распространённые веб-сервисы в мире.

### 6 основных принципов REST (по Филдингу)

| Принцип                          | Описание                                                                | Почему важен в 2026 году                               |
| -------------------------------- | ----------------------------------------------------------------------- | ------------------------------------------------------ |
| **Клиент-сервер**                | Клиент и сервер независимы, могут развиваться отдельно                  | Микросервисы, разные платформы ([[iOS]], Android, Web) |
| **Stateless** (без состояния)    | Каждый запрос содержит всю нужную информацию — сервер не хранит сессию  | Масштабируемость, отказоустойчивость                   |
| **Cacheable** (кэшируемость)     | Ответы могут кэшироваться (заголовки Cache-Control, ETag)               | Снижение нагрузки, ускорение приложений                |
| **Uniform Interface**            | Единый интерфейс: ресурсы через URI, методы HTTP, HATEOAS (опционально) | Предсказуемость, простота интеграции                   |
| **Layered System**               | Многоуровневая архитектура (прокси, балансировщики, CDN)                | Масштабирование, безопасность                          |
| **Code on Demand** (опционально) | Сервер может отправлять код клиенту (например, JavaScript)              | Почти не используется в мобильных API                  |

### [[HTTP]]-методы в RESTful API (самые используемые в 2026)

| Метод                   | Семантика (что делает)          | Идемпотентен? | Безопасен? | Тело запроса | Кэшируется? | Самые частые сценарии       |
| ----------------------- | ------------------------------- | ------------- | ---------- | ------------ | ----------- | --------------------------- |
| [[GET-HTTP\|GET]]       | Получить ресурс / список        | Да            | Да         | Нет          | Да          | Загрузка данных, поиск      |
| [[POST-HTTP\|POST]]     | Создать новый ресурс / действие | Нет           | Нет        | Да           | Нет         | Регистрация, создание поста |
| [[PUT-HTTP\|PUT]]       | Полная замена ресурса           | Да            | Нет        | Да           | Иногда      | Обновление профиля целиком  |
| [[PATCH-HTTP\|PATCH]]   | Частичное обновление            | Да*           | Нет        | Да           | Иногда      | Изменение статуса, лайк     |
| [[DELETE-HTTP\|DELETE]] | Удалить ресурс                  | Да            | Нет        | Иногда       | Нет         | Удаление аккаунта/поста     |

*PATCH идемпотентен при правильной реализации (JSON Patch)

### REST в [[Swift]] / [[iOS]] (реальные примеры 2026)

#### 1. GET-запрос (загрузка списка постов)

```swift
struct Post: Codable {
    let id: Int
    let title: String
    let body: String
}

func fetchPosts() async throws -> [Post] {
    let url = URL(string: "https://jsonplaceholder.typicode.com/posts")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode([Post].self, from: data)
}
```

#### 2. POST-запрос (создание пользователя)

```swift
struct CreateUser: Encodable {
    let name: String
    let email: String
}

func createUser(name: String, email: String) async throws -> User {
    let requestBody = CreateUser(name: name, email: email)
    
    var request = URLRequest(url: URL(string: "https://api.example.com/users")!)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.httpBody = try JSONEncoder().encode(requestBody)
    
    let (data, _) = try await URLSession.shared.data(for: request)
    return try JSONDecoder().decode(User.self, from: data)
}
```

#### 3. PATCH-запрос (частичное обновление)

```swift
struct UpdateUser: Encodable {
    let name: String?
    let bio: String?
}

func updateUser(id: Int, name: String? = nil, bio: String? = nil) async throws {
    let body = UpdateUser(name: name, bio: bio)
    
    var request = URLRequest(url: URL(string: "https://api.example.com/users/\(id)")!)
    request.httpMethod = "PATCH"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.httpBody = try JSONEncoder().encode(body)
    
    let (_, _) = try await URLSession.shared.data(for: request)
}
```

#### 4. DELETE-запрос

```swift
func deletePost(id: Int) async throws {
    var request = URLRequest(url: URL(string: "https://api.example.com/posts/\(id)")!)
    request.httpMethod = "DELETE"
    
    let (_, response) = try await URLSession.shared.data(for: request)
    
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw URLError(.badServerResponse)
    }
}
```

### Лучшие практики REST в Swift 2026

- Всегда используй **HTTPS**
- Используй **Codable** + `JSONEncoder` / `JSONDecoder`
- Предпочитай **async/await** + `URLSession.shared.data(for:)`
- Добавляй заголовки: `Content-Type`, `Accept`, `Authorization`
- Обрабатывай ошибки через `do-try-catch` и `HTTPURLResponse.statusCode`
- Для PATCH — чаще всего используй JSON Merge Patch или JSON Patch
- Кэшируй GET-запросы с помощью `URLCache` или Alamofire/URLSession cache
- Для аутентификации — Bearer Token в `Authorization` header

**Короткое правило**:
> «REST в 2026 — это **GET** для чтения, **POST** для создания, **PATCH** для изменения, **DELETE** для удаления.  
> Всё остальное — через `Codable` + `async/await` + `URLSession`.  
> HTTPS — обязательно, stateless — обязательно, хорошие URI — обязательно.»
