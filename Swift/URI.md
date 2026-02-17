**URI** (Uniform Resource Identifier) — это строка символов, которая **идентифицирует** ресурс в сети (веб-страницу, [[API]], изображение, файл, [[deep link]] и т.д.).

URI — это **общий термин**, который включает в себя два основных подтипа:

| Тип         | Полное название          | Назначение                                         | Пример                                          | Используется в iOS |
| ----------- | ------------------------ | -------------------------------------------------- | ----------------------------------------------- | ------------------ |
| **[[URL]]** | Uniform Resource Locator | Указывает **местоположение** ресурса               | `https://api.example.com/users/123`             | Основной тип       |
| **URN**     | Uniform Resource Name    | Уникально **именует** ресурс (независимо от места) | `urn:uuid:550e8400-e29b-41d4-a716-446655440000` | Редко              |

В 99% случаев в [[iOS]]-разработке вы работаете именно с **URL** (подтип URI).

### Структура URI (по RFC 3986)

URI состоит из следующих частей (слева направо):

```
scheme:[//[userinfo@]host[:port]]path[?query][#fragment]
```

| Компонент       | Обязателен? | Пример в URL                                      | Описание |
|-----------------|-------------|---------------------------------------------------|----------|
| **scheme**      | Да          | `https://`, `http://`, `myapp://`, `file://`      | Протокол или схема (https, custom scheme) |
| **authority**   | Часто       | `user:pass@api.example.com:443`                   | Пользователь:пароль@хост:порт |
| **host**        | Да (если есть authority) | `api.example.com`                                 | Домен или IP-адрес |
| **port**        | Нет         | `:8080`                                           | Порт (по умолчанию: 80 для http, 443 для https) |
| **path**        | Да          | `/users/123/posts`                                | Путь к ресурсу |
| **query**       | Нет         | `?id=123&sort=desc&page=2`                        | Параметры запроса (key=value) |
| **fragment**    | Нет         | `#section-2`                                      | Якорь внутри страницы (клиентская навигация) |

### Реальные примеры разбора URI

| Полный URI                                                           | scheme    | host            | port | path             | query          | fragment |
| -------------------------------------------------------------------- | --------- | --------------- | ---- | ---------------- | -------------- | -------- |
| https://api.example.com/users/123                                    | [[HTTPS]] | api.example.com | —    | /users/123       | —              | —        |
| https://user:pass@api.example.com:8080/search?q=swift&page=2#results | https     | api.example.com | 8080 | /search          | q=swift&page=2 | #results |
| myapp://profile/456?token=abc123                                     | myapp     | —               | —    | /profile/456     | token=abc123   | —        |
| file:///Users/me/Documents/report.pdf                                | file      | —               | —    | /Users/me/...pdf | —              | —        |

### URI в Swift — как работать с ними

В Swift основной тип для работы с URI — это **`URL`** (он реализует почти всё, что нужно).

#### 1. Создание и разбор URL

```swift
let urlString = "https://api.example.com/v1/users/123?sort=desc&page=2#profile"

if let url = URL(string: urlString) {
    print("Scheme:",     url.scheme     ?? "—")      // https
    print("Host:",       url.host       ?? "—")      // api.example.com
    print("Port:",       url.port       ?? "—")      // nil (по умолчанию 443)
    print("Path:",       url.path                )   // /v1/users/123
    print("Query:",      url.query      ?? "—")      // sort=desc&page=2
    print("Fragment:",   url.fragment   ?? "—")      // profile
    
    // Полный разбор query-параметров
    let components = URLComponents(url: url, resolvingAgainstBaseURL: false)
    let queryItems = components?.queryItems ?? []
    
    for item in queryItems {
        print("\(item.name): \(item.value ?? "—")")
    }
    // sort: desc
    // page: 2
}
```

#### 2. Построение URL безопасно ([[URLComponent]])

```swift
var components = URLComponents()
components.scheme = "https"
components.host = "api.example.com"
components.path = "/v1/users"
components.queryItems = [
    URLQueryItem(name: "sort", value: "desc"),
    URLQueryItem(name: "page", value: "2"),
    URLQueryItem(name: "limit", value: "20")
]

if let url = components.url {
    print(url.absoluteString)
    // https://api.example.com/v1/users?sort=desc&page=2&limit=20
}
```

#### 3. Custom URL Scheme ([[deep link]] внутри приложения)

```swift
// Info.plist
// URL Types → Identifier: com.example.app, URL Schemes: myapp

// SceneDelegate
func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
    guard let url = URLContexts.first?.url else { return }
    
    if url.scheme == "myapp" {
        switch url.host {
        case "profile":
            let userID = url.queryParameters?["id"]
            openProfile(userID: userID)
        case "settings":
            openSettings()
        default:
            break
        }
    }
}
```

#### 4. [[Universal Link]] (современный deep linking по HTTPS)

```swift
// SceneDelegate
func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
    guard userActivity.activityType == NSUserActivityTypeBrowsingWeb,
          let url = userActivity.webpageURL else { return }
    
    handleUniversalLink(url: url)
}

func handleUniversalLink(url: URL) {
    let path = url.path
    let params = url.queryParameters
    
    switch path {
    case "/profile":
        if let id = params?["id"] {
            openProfile(userID: id)
        }
    case "/product":
        if let id = params?["id"] {
            openProduct(id: id)
        }
    default:
        break
    }
}
```

#### 5. [[URLSession]] + URI ([[GET-HTTP|GET]] + [[POST-HTTP|POST]] + [[PATCH-HTTP|PATCH]] + [[DELETE-HTTP|DELETE]])

```swift
// GET
func fetchUser(id: String) async throws -> User {
    let url = URL(string: "https://api.example.com/users/\(id)")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(User.self, from: data)
}

// POST
func createPost(title: String) async throws {
    let payload = ["title": title]
    var request = URLRequest(url: URL(string: "https://api.example.com/posts")!)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.httpBody = try JSONEncoder().encode(payload)
    
    let (_, _) = try await URLSession.shared.data(for: request)
}
```

### Лучшие практики работы с URI в Swift 2026

- Всегда используй **[[URLComponent]]** для построения URL — это безопасно и предотвращает ошибки
- Проверяй `url.scheme == "https"` для безопасности
- Для deep linking — **Universal Links** как основной способ, custom scheme — только fallback
- Для API — всегда указывай `Accept: application/json` и `Content-Type: application/json`
- Используй **async/await** + `URLSession.shared.data(for:)` — это стандарт
- Обрабатывай ошибки через `do-try-catch` и проверяй `statusCode`
- Для параметров — используй `URLQueryItem` и `addingPercentEncoding`

**Короткое правило 2026**:
> «URI — это адрес ресурса в интернете.  
> В [[Swift]] работай только с `URL` и `URLComponents`.  
> Для deep linking — **Universal Links** по HTTPS.  
> Для [[API]] — **HTTPS + Codable + async/await + правильные HTTP-методы**.  
> Никогда не конкатенируй строки вручную — это источник багов.»
