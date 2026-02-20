**`URLSession`** — это **центральный [[API]]** в [[Foundation]] для выполнения **всех сетевых операций** в iOS, macOS, tvOS, watchOS и visionOS.

Это **единственный нативный** способ отправлять [[HTTP]]/[[HTTPS]]-запросы, загружать/выгружать файлы, работать с [[WebSocket]] и т.д.

### 1. Три основных типа URLSession

| Тип сессии                  | Когда использовать                              | Особенности и ограничения | Пример создания |
|-----------------------------|--------------------------------------------------|----------------------------|-----------------|
| **shared**                  | Простые запросы без кастомизации                 | Глобальная, нельзя менять конфигурацию | `URLSession.shared` |
| **default**                 | Стандартные запросы с возможностью настройки     | Кэш на диске, cookies, credentials         | `URLSession(configuration: .default)` |
| **ephemeral**               | Высокая приватность, без кэша и cookies          | Всё в памяти, ничего не сохраняется        | `URLSession(configuration: .ephemeral)` |
| **background**              | Загрузка/выгрузка файлов в фоне                  | Работает даже при закрытом приложении      | `URLSession(configuration: backgroundConfig)` |

### 2. Основные конфигурации (URLSessionConfiguration)

| Конфигурация                        | Тип кэша | Cookies | Credentials | Background | Когда использовать |
|-------------------------------------|----------|---------|-------------|------------|--------------------|
| `.default`                          | Диск + память | Да      | Да          | Нет        | Обычные запросы    |
| `.ephemeral`                        | Только память | Нет     | Нет         | Нет        | Приватные запросы  |
| `.background(withIdentifier:)`      | Диск     | Да      | Да          | Да         | Фоновые загрузки   |

### 3. Самые частые методы URLSession (2026)

| Метод / Тип задачи  | Возвращает                  | Синхронность | Когда использовать                            | Пример                                               |
| ------------------- | --------------------------- | ------------ | --------------------------------------------- | ---------------------------------------------------- |
| `data(from:)`       | ([[Data]], [[URLResponse]]) | async        | Простой [[GET-HTTP\|GET]]/[[POST-HTTP\|POST]] | `try await session.data(from: url)`                  |
| `data(for:)`        | `(Data, URLResponse)`       | async        | Запрос с настройками                          | `try await session.data(for: request)`               |
| `upload(for:with:)` | `(Data, URLResponse)`       | async        | Загрузка файла                                | `try await session.upload(for: request, from: data)` |
| `download(from:)`   | `(URL, URLResponse)`        | async        | Скачивание файла                              | `try await session.download(from: url)`              |
| `dataTask(with:)`   | [[URLSessionDataTask]]      | completion   | Legacy / контроль                             | `session.dataTask(with: request) { ... }`            |

### 4. Полные примеры кода (от простого к продвинутому)

#### Пример 1 — Простейший GET ([[async]]/[[await]] + [[Codable]])

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

#### Пример 2 — POST с [[JSON]] и обработкой ошибок

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
    
    let (data, response) = try await URLSession.shared.data(for: request)
    
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw URLError(.badServerResponse)
    }
    
    return try JSONDecoder().decode(Post.self, from: data)
}
```

#### Пример 3 — Кастомная сессия с кэшем и таймаутом

```swift
let config = URLSessionConfiguration.default
config.timeoutIntervalForRequest = 30
config.timeoutIntervalForResource = 60
config.urlCache = URLCache(memoryCapacity: 50 * 1024 * 1024,
                           diskCapacity: 200 * 1024 * 1024,
                           diskPath: "myCache")
config.requestCachePolicy = .returnCacheDataElseLoad

let customSession = URLSession(configuration: config)

func fetchWithCache() async throws -> Data {
    let url = URL(string: "https://example.com/large.json")!
    let (data, _) = try await customSession.data(from: url)
    return data
}
```

#### Пример 4 — Фоновая загрузка файла (background session)

```swift
let config = URLSessionConfiguration.background(withIdentifier: "com.example.background")
let backgroundSession = URLSession(configuration: config, delegate: self, delegateQueue: nil)

func downloadFile(url: URL) {
    let request = URLRequest(url: url)
    let task = backgroundSession.downloadTask(with: request)
    task.resume()
}

// Delegate методы
extension YourClass: URLSessionDownloadDelegate {
    func urlSession(_ session: URLSession, downloadTask: URLSessionDownloadTask, didFinishDownloadingTo location: URL) {
        // Перемещаем файл в Documents
        let documents = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
        let destination = documents.appendingPathComponent(location.lastPathComponent)
        try? FileManager.default.moveItem(at: location, to: destination)
    }
}
```

#### Пример 5 — Загрузка с прогрессом ([[URLSessionTaskDelegate]])

```swift
class DownloadManager: NSObject, URLSessionDownloadDelegate {
    var progressHandler: ((Double) -> Void)?
    
    func urlSession(_ session: URLSession, downloadTask: URLSessionDownloadTask, didWriteData bytesWritten: Int64, totalBytesWritten: Int64, totalBytesExpectedToWrite: Int64) {
        let progress = Double(totalBytesWritten) / Double(totalBytesExpectedToWrite)
        progressHandler?(progress)
    }
    
    func urlSession(_ session: URLSession, downloadTask: URLSessionDownloadTask, didFinishDownloadingTo location: URL) {
        // обработка файла
    }
}

let config = URLSessionConfiguration.default
let session = URLSession(configuration: config, delegate: downloadManager, delegateQueue: nil)
```

### 6. Таблица: когда использовать какой тип сессии

| Сценарий                                 | Рекомендуемая сессия          | Почему                                    |
| ---------------------------------------- | ----------------------------- | ----------------------------------------- |
| Простые GET/POST без кастомизации        | `.shared`                     | Быстро и просто                           |
| [[API]] с авторизацией и заголовками     | `.default` + кастомный config | Гибкость                                  |
| Высокая приватность (без кэша, cookies)  | `.ephemeral`                  | Не сохраняет ничего                       |
| Фоновая загрузка/выгрузка файлов         | `.background`                 | Работает при закрытом приложении          |
| Прогресс загрузки/выгрузки               | Кастомная с delegate          | Доступ к `urlSession(_:didWriteData:...)` |
| Кэширование изображений/статичных данных | `.default` + большой кэш      | Автоматический кэш                        |

### 7. Лучшие практики 2026 года

- Предпочитай **async/await** + `data(from:)` / `data(for:)`  
- Всегда проверяй `response as? HTTPURLResponse` и `statusCode`  
- Используй **Codable** для моделей  
- Для авторизации — Bearer Token в заголовке  
- Для файлов — `downloadTask` или `uploadTask`  
- Для прогресса — `URLSessionTaskDelegate`  
- Для приватности — `.ephemeral`  
- Для фона — `.background` с уникальным identifier  
- Обрабатывай ошибки: 401, 403, 404, 429, 500+

**Короткий девиз**:
> «URLSession — это сердце всей сетевой инфраструктуры iOS.  
> В 2026 году — используй **async/await**, **Codable**, **HTTPS**, проверяй статус-коды и не бойся кастомных конфигураций.»
