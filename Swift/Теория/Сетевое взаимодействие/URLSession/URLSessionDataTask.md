**URLSessionDataTask** — это объект, который выполняет конкретный **[[HTTP]]-запрос** (или другой сетевой таск) и управляет его жизненным циклом.

Это основной «рабочий» объект в классическом [[API]] [[URLSession]] (до [[Combine]] и [[async]]/[[await]]).

### Кратко: что это такое

| Характеристика               | Описание                                                                 |
|------------------------------|--------------------------------------------------------------------------|
| Класс                        | `URLSessionDataTask` : `URLSessionTask` : `NSObject`                     |
| Создаётся через              | `URLSession.dataTask(with:)` или `URLSession.dataTask(with:completionHandler:)` |
| Основная задача              | Загрузить данные по URL (GET/POST/PUT и т.д.) и вернуть `Data`, `URLResponse`, `Error` |
| Асинхронность                | Работает в фоновом потоке, результат приходит через completion или delegate |
| Состояния                    | suspended → running → completed / cancelled                              |
| Отмена                       | `.cancel()` — немедленно прекращает запрос                               |

### Самые актуальные способы использования в 2026 году

#### 1. Классический вариант с completionHandler (всё ещё очень живой)

```swift
let url = URL(string: "https://api.example.com/users")!

let task = URLSession.shared.dataTask(with: url) { data, response, error in
    // Все три параметра могут быть nil одновременно — будьте осторожны
    if let error {
        print("Ошибка сети:", error.localizedDescription)
        return
    }
    
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        print("Сервер ответил не 2xx:", (response as? HTTPURLResponse)?.statusCode ?? -1)
        return
    }
    
    guard let data else {
        print("Данные не получены")
        return
    }
    
    // Декодируем, например, JSON
    do {
        let users = try JSONDecoder().decode([User].self, from: data)
        DispatchQueue.main.async {
            // Обновляем UI
        }
    } catch {
        print("Ошибка декодирования:", error)
    }
}

task.resume()  // ← обязательно!
```

#### 2. Современный вариант с async/await ([[iOS]] 15+ / macOS 12+)

```swift
@MainActor
func fetchUsers() async throws -> [User] {
    let url = URL(string: "https://api.example.com/users")!
    
    let (data, response) = try await URLSession.shared.data(from: url)
    
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw URLError(.badServerResponse)
    }
    
    let users = try JSONDecoder().decode([User].self, from: data)
    return users
}

// Вызов
Task {
    do {
        let users = try await fetchUsers()
        // обновляем UI
    } catch {
        print("Ошибка:", error)
    }
}
```

#### 3. Использование делегатов (если нужен полный контроль)

```swift
class NetworkManager: NSObject, URLSessionDataDelegate {
    private var session: URLSession!
    private var dataTasks: [URLSessionDataTask: (Data?, URLResponse?, Error?) -> Void] = [:]
    
    override init() {
        super.init()
        let config = URLSessionConfiguration.default
        session = URLSession(configuration: config, delegate: self, delegateQueue: nil)
    }
    
    func fetch(url: URL, completion: @escaping (Data?, URLResponse?, Error?) -> Void) {
        let task = session.dataTask(with: url)
        dataTasks[task] = completion
        task.resume()
    }
    
    func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive data: Data) {
        // Можно аккумулировать данные по частям
    }
    
    func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?) {
        if let completion = dataTasks.removeValue(forKey: task as! URLSessionDataTask) {
            completion(nil, task.response, error)
        }
    }
}
```

### Когда использовать URLSessionDataTask в 2026 году

| Ситуация                                      | Рекомендуемый подход 2026                              | Почему |
|-----------------------------------------------|---------------------------------------------------------|--------|
| Новый проект, iOS 15+                         | **async/await + URLSession.shared.data(from:)**         | минимум кода, читаемо, безопасно |
| Проект на iOS 13–14                           | **Combine + dataTaskPublisher**                         | реактивный стиль, операторы |
| Legacy-код / поддержка iOS 12 и ниже          | **Классический dataTask + completionHandler**           | совместимость |
| Нужен прогресс загрузки / upload              | **URLSessionDataTask** + делегаты                       | полный контроль |
| Множество параллельных запросов с отменой     | **URLSessionDataTask** + ручное управление              | можно отменять отдельно |
| Максимальная кастомизация (auth challenge, redirect) | **URLSession** + делегаты                               | полный доступ к delegate-методам |

### Лучшие практики URLSessionDataTask в 2026

- **Предпочитайте async/await**, если минимальная версия ≥ iOS 15  
- **Используйте Combine**, если проект уже на Combine или нужна реактивность  
- **Всегда проверяйте HTTP-статус** — 200...299 не гарантируется  
- **Обрабатывайте ошибки корректно** — `URLError`, `NSError`, `DecodingError`  
- **Используйте `[weak self]`** в completion / sink, если захватываете self  
- **Для больших файлов** — используйте `downloadTask` вместо `dataTask`  
- **Для upload** — `uploadTask` с прогрессом через делегаты  
- **Документируйте** — пишите комментарий:

```swift
/// Асинхронная загрузка пользователей с обработкой ошибок и статус-кодов
func fetchUsers() async throws -> [User] {
    let (data, response) = try await URLSession.shared.data(from: url)
    // ...
}
```

**Короткий итог 2026**:
> `URLSessionDataTask` — это объект, выполняющий **HTTP-запрос** и возвращающий `Data`, `URLResponse`, `Error`.  
> В 2026 году:  
> - **самый современный способ** — `try await URLSession.shared.data(from:)`  
> - **реактивный стиль** — `dataTaskPublisher` + Combine  
> - **классический стиль** — `dataTask(with:completionHandler:)` + `resume()`  
> - это **основа** всей сетевой работы в iOS, даже если вы используете Alamofire / Moya — под капотом часто именно `URLSessionDataTask`  
