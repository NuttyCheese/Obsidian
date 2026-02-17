**RESTful API** — это [[API]], который **строго следует** принципам [[REST]] (по Рою Филдингу):  
- ресурсы идентифицируются [[URI]]  
- действия выполняются через стандартные [[HTTP]]-методы  
- stateless (каждый запрос самодостаточен)  
- кэшируемость, единообразный интерфейс и т.д.

### Основные HTTP-методы в RESTful API

| Метод                   | Семантика (что делает)   | Идемпотентен? | Безопасен? | Тело запроса | Кэшируется? | Пример URI             |
| ----------------------- | ------------------------ | ------------- | ---------- | ------------ | ----------- | ---------------------- |
| [[GET-HTTP\|GET]]       | Получить ресурс / список | Да            | Да         | Нет          | Да          | `/users`, `/posts/123` |
| [[POST-HTTP\|POST]]     | Создать новый ресурс     | Нет           | Нет        | Да           | Нет         | `/users`, `/posts`     |
| [[PUT-HTTP\|PUT]]       | Полная замена ресурса    | Да            | Нет        | Да           | Иногда      | `/users/123`           |
| [[PATCH-HTTP\|PATCH]]   | Частичное обновление     | Да*           | Нет        | Да           | Иногда      | `/users/123`           |
| [[DELETE-HTTP\|DELETE]] | Удалить ресурс           | Да            | Нет        | Иногда       | Нет         | `/users/123`           |

*PATCH идемпотентен при правильной реализации ([[JSON]] Patch, RFC 6902)

### 1. Клиентская часть — работа с RESTful API в Swift (2026)

#### Пример 1: GET-запрос ([async]/[[await]] + [[Codable]]) — самый частый

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

#### Пример 2: POST-запрос (создание ресурса)

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

#### Пример 3: PATCH-запрос (частичное обновление)

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

#### Пример 4: DELETE + обработка ошибок

```swift
func deletePost(id: Int) async throws {
    var request = URLRequest(url: URL(string: "https://jsonplaceholder.typicode.com/posts/\(id)")!)
    request.httpMethod = "DELETE"
    
    let (_, response) = try await URLSession.shared.data(for: request)
    
    guard let httpResponse = response as? HTTPURLResponse else {
        throw URLError(.badServerResponse)
    }
    
    switch httpResponse.statusCode {
    case 200...204: break  // успех
    case 404: throw URLError(.fileDoesNotExist)
    default: throw URLError(.badServerResponse)
    }
}
```

#### Пример 5: Аутентификация (Bearer Token)

```swift
func fetchProtectedData(token: String) async throws -> [Post] {
    var request = URLRequest(url: URL(string: "https://api.example.com/protected")!)
    request.httpMethod = "GET"
    request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
    
    let (data, _) = try await URLSession.shared.data(for: request)
    return try JSONDecoder().decode([Post].self, from: data)
}
```

#### Пример 6: [[multipart or form-data]] (отправка файла)

```swift
func uploadImage(_ imageData: Data, filename: String) async throws {
    let boundary = "Boundary-\(UUID().uuidString)"
    var request = URLRequest(url: URL(string: "https://api.example.com/upload")!)
    request.httpMethod = "POST"
    request.setValue("multipart/form-data; boundary=\(boundary)", forHTTPHeaderField: "Content-Type")
    
    var body = Data()
    
    body.append("--\(boundary)\r\n".data(using: .utf8)!)
    body.append("Content-Disposition: form-data; name=\"file\"; filename=\"\(filename)\"\r\n".data(using: .utf8)!)
    body.append("Content-Type: image/jpeg\r\n\r\n".data(using: .utf8)!)
    body.append(imageData)
    body.append("\r\n".data(using: .utf8)!)
    
    body.append("--\(boundary)--\r\n".data(using: .utf8)!)
    
    request.httpBody = body
    
    let (_, _) = try await URLSession.shared.data(for: request)
}
```

### Лучшие практики RESTful API в Swift 2026

- Всегда **[[HTTPS]]**
- Модели — **[[Codable]]**
- Предпочитай **async/await + [[URLSession]].shared.data(for:)**
- Заголовки: `Accept: application/json`, `Content-Type: application/json`
- Обрабатывай статус-коды (200–299 — успех, 400–499 — клиентская ошибка, 500+ — серверная)
- Для PATCH — чаще JSON Merge Patch или JSON Patch
- Кэшируй GET-запросы (`URLCache` или `URLSessionConfiguration`)
- Аутентификация — Bearer Token в `Authorization` header
- Логируй ошибки с `localizedDescription` + `debugDescription`
- Используй **URLComponents** для безопасного построения URL + query

**Короткое правило**:
> «RESTful — это когда ты используешь **GET** для чтения, **POST** для создания, **PATCH** для изменения, **DELETE** для удаления.  
> В Swift 2026 — делай всё через **Codable** + **async/await** + **URLSession**.  
> HTTPS — обязательно, stateless — обязательно, хорошие URI — обязательно.»
