**Alamofire** — это популярная высокоуровневая [[Swift]]-библиотека для работы с сетью. Она предоставляет удобный [[API]] поверх `URLSession`, упрощая выполнение [[HTTP]]-запросов, загрузку/отправку файлов, обработку ответов, декодирование [[JSON]], манипуляции с заголовками, очередями и валидацию.

### **К какому стеку относится**

- **Сторонний фреймворк (3rd-party)**
    
- Построен поверх **[[Foundation]] → [[URLSession]]**
    
- Используется в [[iOS]]/macOS/tvOS/watchOS проектах на Swift
    
- Обычно работает в связке с **[[UIKit]] / [[SwiftUI]]** в приложениях
    

---

## **1. Простой [[GET-HTTP|GET]]-запрос**

```swift
import Alamofire

AF.request("https://api.example.com/users")
    .responseJSON { response in
        switch response.result {
        case .success(let json):
            print("Ответ:", json)
        case .failure(let error):
            print("Ошибка:", error.localizedDescription)
        }
    }
```

**Комментарий:** самый простой запрос, без параметров.

---

## **2. [[GET-HTTP|GET]]-запрос с параметрами**

```swift
let params: Parameters = [
    "page": 1,
    "limit": 20
]

AF.request("https://api.example.com/items", parameters: params)
    .responseDecodable(of: [Item].self) { response in
        switch response.result {
        case .success(let items):
            print("Получено:", items)
        case .failure(let error):
            print("Ошибка:", error)
        }
    }
```

**Комментарий:** безопасное декодирование через [[Decodable]].

---

## **3. [[POST-HTTP|POST]]-запрос с JSON-телом**

```swift
let body: [String: Any] = [
    "email": "test@mail.com",
    "password": "123456"
]

AF.request("https://api.example.com/login",
           method: .post,
           parameters: body,
           encoding: JSONEncoding.default)
    .responseDecodable(of: LoginResponse.self) { response in
        switch response.result {
        case .success(let model):
            print("Успех:", model)
        case .failure(let error):
            print("Ошибка:", error)
        }
    }
```

**Комментарий:** отправляем JSON, используем [[JSONEncoding]].

---

## **4. Загрузка файла ([[multipart or form-data]])**

```swift
AF.upload(multipartFormData: { formData in
    if let data = UIImage(named: "photo")?.jpegData(compressionQuality: 0.8) {
        formData.append(data,
                        withName: "file",
                        fileName: "photo.jpg",
                        mimeType: "image/jpeg")
    }
},
to: "https://api.example.com/upload")
.uploadProgress { progress in
    print("Прогресс:", progress.fractionCompleted)
}
.responseJSON { response in
    print("Загрузка завершена:", response)
}
```

**Комментарий:** удобный [[API]] для загрузки изображений или видео.

---

## **5. Работа с кастомными заголовками + интерсептор**

```swift
class AuthInterceptor: RequestInterceptor {

    func adapt(_ urlRequest: URLRequest,
               for session: Session,
               completion: @escaping (Result<URLRequest, Error>) -> Void) {

        var request = urlRequest
        request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        completion(.success(request))
    }
}

let session = Session(interceptor: AuthInterceptor())

session.request("https://api.example.com/profile")
    .responseDecodable(of: Profile.self) { response in
        print(response)
    }
```

**Комментарий:** интерсептор автоматически добавляет токен ко всем запросам.

---

## **6. Свой `Session` с кастомной конфигурацией**

```swift
let configuration = URLSessionConfiguration.default
configuration.timeoutIntervalForRequest = 15
configuration.waitsForConnectivity = true

let session = Session(configuration: configuration)

session.request("https://api.example.com/slow")
    .validate(statusCode: 200..<300)
    .responseString { response in
        print(response)
    }
```

**Комментарий:** тонкая настройка под нужды проекта.

---

## **7. Параллельное выполнение запросов + обработка через [[DispatchGroup]]**

```swift
let group = DispatchGroup()

var first: User?
var second: [Item]?

group.enter()
AF.request("https://api.example.com/user")
    .responseDecodable(of: User.self) { response in
        first = try? response.result.get()
        group.leave()
    }

group.enter()
AF.request("https://api.example.com/items")
    .responseDecodable(of: [Item].self) { response in
        second = try? response.result.get()
        group.leave()
    }

group.notify(queue: .main) {
    print("Оба запроса завершены")
    print(first as Any)
    print(second as Any)
}
```

**Комментарий:** типичная логика для загрузки нескольких ресурсов.
