#network #Swift 
**REST API** (Representational State Transfer Application Programming Interface) — это архитектурный стиль построения веб-сервисов, основанный на принципах [[HTTP]] и ресурсов.

### Основные принципы REST (кратко)

- **Клиент-сервер** — независимые части
- **Stateless** — каждый запрос самодостаточен
- **Cacheable** — ответы можно кэшировать
- **Uniform Interface** — ресурсы через URI + стандартные HTTP-методы
- **Layered System** — многоуровневая архитектура
- **Code on Demand** (опционально)

### Самые используемые HTTP-методы в REST (2026)

| Метод                   | Что делает                      | Тело запроса | Идемпотентен? | Безопасен? | Кэшируется? | Пример URI             |
| ----------------------- | ------------------------------- | ------------ | ------------- | ---------- | ----------- | ---------------------- |
| [[GET-HTTP\|GET]]       | Получить ресурс / список        | Нет          | Да            | Да         | Да          | `/users`, `/posts/123` |
| [[POST-HTTP\|POST]]     | Создать новый ресурс / действие | Да           | Нет           | Нет        | Нет         | `/users`, `/posts`     |
| [[PUT-HTTP\|PUT]]       | Полная замена ресурса           | Да           | Да            | Нет        | Иногда      | `/users/123`           |
| [[PATCH-HTTP\|PATCH]]   | Частичное обновление            | Да           | Да*           | Нет        | Иногда      | `/users/123`           |
| [[DELETE-HTTP\|DELETE]] | Удалить ресурс                  | Иногда       | Да            | Нет        | Нет         | `/users/123`           |

*PATCH идемпотентен при правильной реализации (JSON Patch, RFC 6902)

### Инструменты для работы с REST API в Swift (2026)

| Библиотека / инструмент              | Уровень   | Современность | Когда использовать                          |
| ------------------------------------ | --------- | ------------- | ------------------------------------------- |
| **[[URLSession]]** (нативный)        | Нативный  | ★★★★★         | Всегда по умолчанию, особенно с async/await |
| **[[Alamofire]]**                    | Сторонняя | ★★★★☆         | Удобный синтаксис, interceptors, multipart  |
| **[[Combine]] + URLSession**         | Нативный  | ★★★★☆         | Реактивный стиль, подписка на изменения     |
| **[[async]]/[[await]] + URLSession** | Нативный  | ★★★★★         | Самый рекомендуемый в 2026                  |
| **Swift Concurrency**                | Нативный  | ★★★★★         | Современные API с [[throws]] + [[async]]    |

### Примеры кода (от простого к продвинутому)

#### 1. GET-запрос ([[async]]/[[await]] + [[Codable]]) — самый популярный в 2026

```swift
struct Post: Codable {
    let id: Int
    let title: String
    let body: String
    let userId: Int
}

func fetchPosts() async throws -> [Post] {
    let url = URL(string: "https://jsonplaceholder.typicode.com/posts")!
    let (data, response) = try await URLSession.shared.data(from: url)
    
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw URLError(.badServerResponse)
    }
    
    return try JSONDecoder().decode([Post].self, from: data)
}

// Использование
Task {
    do {
        let posts = try await fetchPosts()
        print("Получено постов: \(posts.count)")
    } catch {
        print("Ошибка:", error.localizedDescription)
    }
}
```

#### 2. POST-запрос с JSON (создание ресурса)

```swift
struct CreatePost: Encodable {
    let title: String
    let body: String
    let userId: Int
}

func createPost(title: String, body: String, userId: Int) async throws -> Post {
    let payload = CreatePost(title: title, body: body, userId: userId)
    
    var request = URLRequest(url: URL(string: "https://jsonplaceholder.typicode.com/posts")!)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.httpBody = try JSONEncoder().encode(payload)
    
    let (data, _) = try await URLSession.shared.data(for: request)
    return try JSONDecoder().decode(Post.self, from: data)
}
```

#### 3. PATCH-запрос (частичное обновление)

```swift
struct UpdatePost: Encodable {
    let title: String?
    let body: String?
}

func updatePost(id: Int, title: String? = nil, body: String? = nil) async throws {
    let payload = UpdatePost(title: title, body: body)
    
    var request = URLRequest(url: URL(string: "https://jsonplaceholder.typicode.com/posts/\(id)")!)
    request.httpMethod = "PATCH"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.httpBody = try JSONEncoder().encode(payload)
    
    let (_, response) = try await URLSession.shared.data(for: request)
    
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw URLError(.badServerResponse)
    }
}
```

#### 4. DELETE-запрос

```swift
func deletePost(id: Int) async throws {
    var request = URLRequest(url: URL(string: "https://jsonplaceholder.typicode.com/posts/\(id)")!)
    request.httpMethod = "DELETE"
    
    let (_, response) = try await URLSession.shared.data(for: request)
    
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw URLError(.badServerResponse)
    }
}
```

#### 5. Алгоритм обработки ответа (рекомендуемый шаблон)

```swift
func performRequest<T: Decodable>(_ request: URLRequest) async throws -> T {
    let (data, response) = try await URLSession.shared.data(for: request)
    
    guard let httpResponse = response as? HTTPURLResponse else {
        throw URLError(.badServerResponse)
    }
    
    switch httpResponse.statusCode {
    case 200...299:
        return try JSONDecoder().decode(T.self, from: data)
    case 400...499:
        throw URLError(.badURL) // или кастомная ошибка
    case 500...599:
        throw URLError(.badServerResponse)
    default:
        throw URLError(.unknown)
    }
}
```

### Лучшие практики REST в Swift 2026

- Используй **HTTPS** всегда
- Все модели — **Codable**
- Предпочитай **async/await + URLSession** (нативно и современно)
- Добавляй заголовки: `Accept: application/json`, `Content-Type: application/json`
- Обрабатывай статус-коды (200–299 — успех, 400–499 — клиентская ошибка, 500+ — серверная)
- Для PATCH — чаще всего JSON Merge Patch или JSON Patch
- Кэшируй GET-запросы (`URLCache` или `URLSessionConfiguration`)
- Для аутентификации — Bearer Token в `Authorization` header
- Логируй ошибки с `localizedDescription` и `debugDescription`

**Короткое правило**:
> «REST в 2026 — это **GET** для чтения, **POST** для создания, **PATCH** для изменения, **DELETE** для удаления.  
> Всё остальное — через `Codable` + `async/await` + `URLSession`.  
> HTTPS — обязательно, stateless — обязательно, хорошие URI — обязательно.»
