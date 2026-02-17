**JSON** (JavaScript Object Notation) — это **лёгкий**, **текстовый**, **универсальный** формат обмена данными, который стал де-факто стандартом в веб- и мобильной разработке.

### Что такое JSON на самом деле

JSON — это просто **структурированный текст**, который:

- основан на синтаксисе JavaScript (но независим от языка),
- полностью самодостаточен,
- читаем человеком и машиной,
- поддерживается практически всеми современными языками программирования.

### Основные типы данных в JSON

| Тип в JSON | Пример в JSON                         | Эквивалент в Swift             |
| ---------- | ------------------------------------- | ------------------------------ |
| объект     | `{ "name": "Анна", "age": 28 }`       | [[struct]] / [[Dictionary]]    |
| массив     | `[1, 2, 3]` или `["apple", "banana"]` | [[Array]]                      |
| строка     | `"Hello"`                             | [[String]]                     |
| число      | `42` или `3.14`                       | [[Int]], [[Double]], [[Float]] |
| булево     | `true` / `false`                      | [[Bool]]                       |
| null       | `null`                                | [[nil]]                        |

### Для чего используется JSON (самые частые сценарии в 2026 году)

1. **[[API]] ([[REST]], [[GraphQL]], [[WebSocket]])**  
   Почти 95% всех современных API возвращают и принимают данные в [[JSON]].

2. **Конфигурация приложений**  
   Файлы настроек, feature flags, A/B-тесты (часто в связке с Remote Config).

3. **Локальное хранение**  
   Кэширование ответов, сохранение состояния ([[Swift/Теория/Хранение данных/UserDefaults]] + [[JSONEncoder]]).

4. **Обмен данными между фронтендом и бэкендом**  
   Мобильное приложение ↔ сервер.

5. **Межсервисное общение**  
   Микросервисы, Serverless (Cloud Functions, Lambda).

6. **Экспорт/импорт данных**  
   Резервные копии, миграции, отчёты.

7. **[[SwiftUI]] + [[Combine]] / [[async]]**  
   Загрузка данных → декодирование → обновление UI.

### Примеры использования JSON в Swift

#### 1. Декодирование ответа API (самый частый случай)

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

#### 2. Отправка данных на сервер ([[POST-HTTP|POST]] + JSON)

```swift
struct CreatePost: Encodable {
    let title: String
    let body: String
    let userId: Int
}

func createPost() async throws {
    let post = CreatePost(title: "Новый пост", body: "Текст...", userId: 1)
    
    var request = URLRequest(url: URL(string: "https://jsonplaceholder.typicode.com/posts")!)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.httpBody = try JSONEncoder().encode(post)
    
    let (_, _) = try await URLSession.shared.data(for: request)
}
```

#### 3. Сохранение и загрузка локально (UserDefaults + JSON)

```swift
struct Settings: Codable {
    var darkMode: Bool
    var notificationsEnabled: Bool
}

func saveSettings(_ settings: Settings) {
    if let data = try? JSONEncoder().encode(settings) {
        UserDefaults.standard.set(data, forKey: "settings")
    }
}

func loadSettings() -> Settings? {
    guard let data = UserDefaults.standard.data(forKey: "settings") else { return nil }
    return try? JSONDecoder().decode(Settings.self, from: data)
}
```

#### 4. Работа с датами (самая частая проблема)

```swift
let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .iso8601  // или .formatted(...)

let encoder = JSONEncoder()
encoder.dateEncodingStrategy = .iso8601
```

### Почему JSON до сих пор доминирует в 2026 году

- **Универсальность** — понимают все языки и платформы
- **Читаемость** — человек легко видит структуру
- **Простота** — минимум boilerplate с `Codable`
- **Экосистема** — JSON Schema, OpenAPI, GraphQL, REST — всё построено вокруг JSON
- **Производительность** — достаточно быстр для большинства задач (хотя Protobuf быстрее)

### Короткий итог

**JSON** — это **текстовый формат** для структурированных данных  
**Codable** — делает работу с ним почти бесплатной  
**В 2026 году** почти любой мобильный/веб-API использует JSON как основной формат.

**Главное правило**:
> «Если тебе нужно передать или сохранить структурированные данные — по умолчанию бери **JSON + Codable**.  
> Всё остальное (Protobuf, MessagePack, BSON) — только если есть реальная потребность в производительности или размере.»
