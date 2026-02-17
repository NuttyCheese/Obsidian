## 📘 Определение

**Moya** — это [[Swift]]-библиотека для **сетевого слоя**, построенная поверх **[[Alamofire]]**.  
Она предоставляет **структурированный и типобезопасный способ работы с [[API]]**, используя `TargetType` для описания каждого эндпоинта, а также интеграцию с [[Combine]], [[RxSwift]] и другими реактивными подходами.  
Относится к **Network layer → Alamofire / [[Swift/Теория/Сетевое взаимодействие/REST API]]**.

---

## 🔹 Примеры кода

### 1. Установка Moya через [[CocoaPods]]

```ruby
pod 'Moya'
```

или через Swift Package Manager:

```swift
.package(url: "https://github.com/Moya/Moya.git", from: "15.0.0")
```

---

### 2. Определение TargetType

```swift
import Moya

enum MyAPI {
    case getUsers
    case createUser(name: String)
}

extension MyAPI: TargetType {
    var baseURL: URL { URL(string: "https://example.com")! }
    
    var path: String {
        switch self {
        case .getUsers: return "/users"
        case .createUser: return "/users"
        }
    }
    
    var method: Moya.Method {
        switch self {
        case .getUsers: return .get
        case .createUser: return .post
        }
    }
    
    var task: Task {
        switch self {
        case .getUsers:
            return .requestPlain
        case .createUser(let name):
            return .requestParameters(parameters: ["name": name], encoding: JSONEncoding.default)
        }
    }
    
    var headers: [String: String]? {
        ["Content-Type": "application/json"]
    }
    
    var sampleData: Data {
        return Data()
    }
}
```

---

### 3. Создание провайдера и вызов запроса

```swift
let provider = MoyaProvider<MyAPI>()

provider.request(.getUsers) { result in
    switch result {
    case .success(let response):
        print("Ответ:", String(data: response.data, encoding: .utf8) ?? "")
    case .failure(let error):
        print("Ошибка:", error)
    }
}
```

---

### 4. Использование с [[Codable]]

```swift
struct User: Codable {
    let id: Int
    let name: String
}

provider.request(.getUsers) { result in
    switch result {
    case .success(let response):
        let users = try? JSONDecoder().decode([User].self, from: response.data)
        print(users ?? [])
    case .failure(let error):
        print(error)
    }
}
```

---

### 5. Использование с [[Combine]]

```swift
import Combine

let provider = MoyaProvider<MyAPI>()
var cancellables = Set<AnyCancellable>()

provider.requestPublisher(.getUsers)
    .map([User].self)
    .sink { completion in
        print(completion)
    } receiveValue: { users in
        print(users)
    }
    .store(in: &cancellables)
```
