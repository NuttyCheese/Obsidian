## 📘 Определение

**`JSONEncoding`** — объект в [[Alamofire]], который отвечает за **кодирование данных в JSON для [[HTTP]]-запросов**.  
Используется для **[[POST-HTTP|POST]], [[PUT-HTTP|PUT]], [[PATCH-HTTP|PATCH]]** запросов, когда нужно отправить словари или модели как [[JSON]] в теле запроса.  
Относится к **[[Alamofire]] → Network layer**.

---

## 🔹 Примеры кода

### 1. Простое кодирование словаря в JSON

```swift
import Alamofire

let parameters: [String: Any] = ["name": "John", "age": 30]

AF.request("https://example.com/api",
           method: .post,
           parameters: parameters,
           encoding: JSONEncoding.default)
  .responseJSON { response in
      print(response)
  }
```

---

### 2. Отправка JSON с кастомными заголовками

```swift
let headers: HTTPHeaders = [
    "Authorization": "Bearer token",
    "Content-Type": "application/json"
]

AF.request("https://example.com/api",
           method: .post,
           parameters: parameters,
           encoding: JSONEncoding.default,
           headers: headers)
  .responseJSON { response in
      print(response)
  }
```

---

### 3. Отправка [[Codable]] модели

```swift
struct User: Codable {
    let name: String
    let age: Int
}

let user = User(name: "Alice", age: 25)

AF.request("https://example.com/api",
           method: .post,
           parameters: try? user.asDictionary(),
           encoding: JSONEncoding.default)
  .responseJSON { response in
      print(response)
  }

extension Encodable {
    func asDictionary() throws -> [String: Any] {
        let data = try JSONEncoder().encode(self)
        return try JSONSerialization.jsonObject(with: data) as? [String: Any] ?? [:]
    }
}
```

---

### 4. Использование `prettyPrinted` JSON

```swift
let encoding = JSONEncoding(options: .prettyPrinted)

AF.request("https://example.com/api",
           method: .post,
           parameters: parameters,
           encoding: encoding)
  .responseJSON { response in
      print(response)
  }
```

---

### 5. JSONEncoding в PUT запросе

```swift
AF.request("https://example.com/api/123",
           method: .put,
           parameters: parameters,
           encoding: JSONEncoding.default)
  .responseJSON { response in
      print(response)
  }
```
