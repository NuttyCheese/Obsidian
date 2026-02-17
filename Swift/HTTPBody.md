**`httpBody`** — это свойство `URLRequest`, которое содержит **тело HTTP-запроса** (payload).  
Используется в основном для **POST**, **PUT**, **PATCH** и иногда **DELETE** запросов, чтобы передать данные на сервер: [[JSON]], формы, файлы, бинарные данные и т.д.

### Когда и для каких методов нужно httpBody

| HTTP-метод | Должен иметь тело? | Чаще всего используется для | Примечание |
|------------|---------------------|-----------------------------|------------|
| **GET**    | Нет (по стандарту)  | —                           | Некоторые серверы игнорируют, но RFC запрещает |
| **POST**   | Да                  | Создание ресурса, формы, авторизация | Самый частый |
| **PUT**    | Да                  | Полная замена ресурса       | Полный объект |
| **PATCH**  | Да                  | Частичное обновление        | JSON Patch, merge-patch |
| **DELETE** | Иногда (редко)      | Удаление с параметрами      | Например, удаление списка ID |
| **HEAD**   | Нет                 | —                           | Только заголовки |
| **OPTIONS**| Нет                 | —                           | CORS preflight |

### Основные форматы тела запроса (Content-Type)

| Content-Type                          | Когда использовать                          | Пример кода |
|---------------------------------------|---------------------------------------------|-------------|
| `application/json`                    | Передача структурированных данных (самый частый) | JSONEncoder |
| `application/x-www-form-urlencoded`   | Классические формы (логин, поиск)          | URL-encoded строка |
| `multipart/form-data`                 | Отправка файлов + полей формы               | `URLSession` + boundary |
| `application/octet-stream`            | Бинарные файлы (изображения, PDF)           | `Data` напрямую |
| `text/plain`                          | Простой текст                               | Редко |

### Примеры кода (от простого к продвинутому)

#### 1. Простой POST с текстом

```swift
var request = URLRequest(url: URL(string: "https://example.com/echo")!)
request.httpMethod = "POST"
request.httpBody = "Hello from Swift!".data(using: .utf8)

URLSession.shared.dataTask(with: request) { data, response, error in
    if let data, let str = String(data: data, encoding: .utf8) {
        print("Ответ сервера:", str)
    }
}.resume()
```

#### 2. POST с JSON (Codable + async/await — самый современный способ)

```swift
struct CreatePostRequest: Encodable {
    let title: String
    let body: String
    let userId: Int
}

func createPost() async throws -> Post {
    let requestBody = CreatePostRequest(title: "Мой пост", body: "Текст...", userId: 1)
    
    var request = URLRequest(url: URL(string: "https://jsonplaceholder.typicode.com/posts")!)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    
    request.httpBody = try JSONEncoder().encode(requestBody)
    
    let (data, _) = try await URLSession.shared.data(for: request)
    return try JSONDecoder().decode(Post.self, from: data)
}

// Использование
Task {
    do {
        let post = try await createPost()
        print("Создан пост с ID:", post.id)
    } catch {
        print("Ошибка:", error.localizedDescription)
    }
}
```

#### 3. Отправка формы (x-www-form-urlencoded)

```swift
var request = URLRequest(url: URL(string: "https://example.com/login")!)
request.httpMethod = "POST"
request.setValue("application/x-www-form-urlencoded", forHTTPHeaderField: "Content-Type")

let params = [
    "username": "john",
    "password": "secret123",
    "remember_me": "true"
]

let bodyString = params.map { "\($0.key)=\($0.value.addingPercentEncoding(withAllowedCharacters: .urlQueryAllowed) ?? "")" }
                       .joined(separator: "&")

request.httpBody = bodyString.data(using: .utf8)
```

#### 4. Отправка файла ([[multipart or form-data]])

```swift
func uploadImage(_ imageData: Data, filename: String) async throws {
    let boundary = "Boundary-\(UUID().uuidString)"
    var request = URLRequest(url: URL(string: "https://example.com/upload")!)
    request.httpMethod = "POST"
    request.setValue("multipart/form-data; boundary=\(boundary)", forHTTPHeaderField: "Content-Type")
    
    var body = Data()
    
    // Добавляем файл
    body.append("--\(boundary)\r\n".data(using: .utf8)!)
    body.append("Content-Disposition: form-data; name=\"file\"; filename=\"\(filename)\"\r\n".data(using: .utf8)!)
    body.append("Content-Type: image/jpeg\r\n\r\n".data(using: .utf8)!)
    body.append(imageData)
    body.append("\r\n".data(using: .utf8)!)
    
    // Дополнительные поля (опционально)
    body.append("--\(boundary)\r\n".data(using: .utf8)!)
    body.append("Content-Disposition: form-data; name=\"userId\"\r\n\r\n".data(using: .utf8)!)
    body.append("123".data(using: .utf8)!)
    body.append("\r\n".data(using: .utf8)!)
    
    body.append("--\(boundary)--\r\n".data(using: .utf8)!)
    
    request.httpBody = body
    
    let (data, _) = try await URLSession.shared.data(for: request)
    print("Ответ:", String(data: data, encoding: .utf8) ?? "")
}
```

#### 5. Отправка бинарных данных (например, изображение напрямую)

```swift
let imageData = UIImage(systemName: "photo")!.jpegData(compressionQuality: 0.8)!
var request = URLRequest(url: URL(string: "https://example.com/upload")!)
request.httpMethod = "POST"
request.setValue("image/jpeg", forHTTPHeaderField: "Content-Type")
request.httpBody = imageData

URLSession.shared.dataTask(with: request) { data, response, error in
    print("Загружено")
}.resume()
```

#### 6. Автоматическая сериализация массива объектов

```swift
struct Todo: Codable {
    let title: String
    let completed: Bool
}

let todos = [
    Todo(title: "Купить молоко", completed: false),
    Todo(title: "Позвонить маме", completed: true)
]

var request = URLRequest(url: URL(string: "https://example.com/todos")!)
request.httpMethod = "POST"
request.setValue("application/json", forHTTPHeaderField: "Content-Type")
request.httpBody = try? JSONEncoder().encode(todos)
```

### Лучшие практики 2026 года

- Используй **async/await + URLSession.shared.data(for:)** — это стандарт
- Всегда устанавливай `Content-Type` явно
- Для JSON — `JSONEncoder` + `Codable` (не `JSONSerialization`)
- Для форм — `application/x-www-form-urlencoded` или `multipart/form-data`
- Для файлов — всегда `multipart/form-data`
- Обрабатывай ошибки через `do-try-catch`
- Используй `URLComponents` для безопасного построения query + path
- В продакшене — добавляй `timeoutInterval`, `cachePolicy`, авторизацию

**Короткое правило**:
> «httpBody нужен почти всегда, когда метод не GET/HEAD/OPTIONS.  
> [[JSON]] + [[Codable]] + [[async]]/[[await]] — золотой стандарт в 2026 году.»
