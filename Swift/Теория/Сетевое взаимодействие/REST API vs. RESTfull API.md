**REST API** и **RESTful API** часто путают или используют как синонимы, но между ними есть **небольшая, но важная разница** в строгости соблюдения принципов.

### Краткое определение

| Термин              | Что это значит                                                                                         | Строгость соблюдения REST-принципов | Пример в реальной жизни                                          |
| ------------------- | ------------------------------------------------------------------------------------------------------ | ----------------------------------- | ---------------------------------------------------------------- |
| **[[Swift/Теория/Сетевое взаимодействие/REST API]]**    | Любой [[API]], который **вдохновлён** принципами [[REST]] (часто использует [[HTTP]]-методы и ресурсы) | Средняя / низкая                    | Многие внутренние API компаний, старые проекты                   |
| **[[RESTful API]]** | API, который **строго следует** всем принципам REST (по Филдингу)                                      | Высокая                             | Современные публичные API (Stripe, [[GitHub]], Twitter/X API v2) |

### 6 принципов REST (по Рою Филдингу, 2000)

1. **Клиент-сервер** — разделение клиента и сервера  
2. **Stateless** — каждый запрос самодостаточен (нет сессий на сервере)  
3. **Cacheable** — ответы можно кэшировать (заголовки Cache-Control, ETag)  
4. **Uniform Interface** — единый интерфейс:  
   - ресурсы идентифицируются URI  
   - манипуляции через стандартные HTTP-методы  
   - представления (representations)  
   - HATEOAS (опционально, редко в 2026)  
5. **Layered System** — многоуровневая архитектура (прокси, балансировщики, CDN)  
6. **Code on Demand** (опционально) — сервер может отправлять код клиенту  

**REST API** может нарушать 1–2 принципа и всё равно называться REST.  
**RESTful API** — строго следует всем (или почти всем).

### Сравнение REST API и RESTful API

| Критерий                  | REST API (нестрогий)                     | RESTful API (строгий)                       | Пример нарушения в REST API |
|---------------------------|------------------------------------------|---------------------------------------------|-----------------------------|
| Stateless                 | Может хранить сессию на сервере          | Нет состояния между запросами               | Использование cookies + сессий |
| Uniform Interface         | Может использовать нестандартные методы  | Только GET/POST/PUT/PATCH/DELETE            | Метод `GET /users/create`   |
| Ресурсы через URI         | Может использовать действия в URI        | Ресурсы: `/users/123`, действия — методы    | `/users/delete/123`         |
| HATEOAS                   | Почти никогда                            | Желательно (ссылки в ответах)               | Отсутствие гиперссылок      |
| Кэширование               | Часто игнорируется                       | Заголовки Cache-Control, ETag               | Нет ETag / Cache-Control    |

### Примеры кода в [[Swift]] (как работать с REST/RESTful API)

#### 1. GET — загрузка списка (самый частый запрос)

```swift
struct Post: Codable {
    let id: Int
    let title: String
    let body: String
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
```

#### 2. POST — создание ресурса (RESTful)

```swift
struct CreatePost: Encodable {
    let title: String
    let body: String
    let userId: Int
}

func createPost() async throws -> Post {
    let payload = CreatePost(title: "Новый пост", body: "Текст...", userId: 1)
    
    var request = URLRequest(url: URL(string: "https://jsonplaceholder.typicode.com/posts")!)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.httpBody = try JSONEncoder().encode(payload)
    
    let (data, _) = try await URLSession.shared.data(for: request)
    return try JSONDecoder().decode(Post.self, from: data)
}
```

#### 3. PATCH — частичное обновление (строго RESTful)

```swift
struct UpdatePost: Encodable {
    let title: String?
    let body: String?
}

func updatePost(id: Int) async throws {
    let payload = UpdatePost(title: "Обновлённый заголовок", body: nil)
    
    var request = URLRequest(url: URL(string: "https://jsonplaceholder.typicode.com/posts/\(id)")!)
    request.httpMethod = "PATCH"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.httpBody = try JSONEncoder().encode(payload)
    
    let (_, _) = try await URLSession.shared.data(for: request)
}
```

#### 4. DELETE — удаление ресурса

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

### Лучшие практики REST API в Swift 2026

- Всегда **HTTPS**
- Используй **Codable** + `JSONEncoder`/`JSONDecoder`
- Предпочитай **async/await** + `URLSession.shared.data(for:)`
- Заголовки: `Accept: application/json`, `Content-Type: application/json`
- Обрабатывай статус-коды (200–299 — успех, 400–499 — клиентская ошибка, 500+ — серверная)
- Для PATCH — чаще JSON Merge Patch или JSON Patch
- Кэшируй GET-запросы (`URLCache` или `URLSessionConfiguration`)
- Аутентификация — Bearer Token в `Authorization` header
- Логируй ошибки с `localizedDescription` + `debugDescription`

**Короткое правило 2026**:
> «RESTful — это когда API **строго** использует HTTP-методы для действий над ресурсами и следует stateless.  
> В реальной жизни почти все API называют себя REST, но **настоящий RESTful** — это высокий уровень зрелости (уровень 3 по модели Ричардсона).  
> В [[Swift]] 2026 — используй **[[GET-HTTP|GET]]/[[POST-HTTP|POST]]/[[PATCH-HTTP|PATCH]]/[[DELETE-HTTP|DELETE]]** + **[[Codable]]** + **[[async]]/[[await]]** — это золотой стандарт.»
