#combine #reactive #Swift 
## 📘 Определение

**`tryMap`** — оператор в **Combine** (и аналогично в [[RxSwift]] через [[map]] с [[try]]), который **преобразует элементы потока**, но при этом может **выбрасывать ошибку**.  
Если возникает ошибка, поток **завершается с `Failure`**.  
Относится к **[[Combine]] / Reactive Programming**.

---

## 🔹 Примеры кода

### 1. Простейшее использование `tryMap`

```swift
import Combine

let numbers = [1, 2, 3, 0]
let publisher = numbers.publisher

publisher
    .tryMap { value -> Int in
        guard value != 0 else {
            throw NSError(domain: "ZeroError", code: 1, userInfo: nil)
        }
        return 10 / value
    }
    .sink(receiveCompletion: { completion in
        print("Completion:", completion)
    }, receiveValue: { value in
        print("Value:", value)
    })
```

---

### 2. Преобразование строк в числа с `tryMap`

```swift
let strings = ["10", "20", "abc"]
let publisher2 = strings.publisher

publisher2
    .tryMap { str -> Int in
        guard let number = Int(str) else {
            throw NSError(domain: "ParseError", code: 2, userInfo: nil)
        }
        return number * 2
    }
    .sink(receiveCompletion: { print($0) },
          receiveValue: { print($0) })
```

---

### 3. Работа с [[API]] и [[JSON]]

```swift
struct User: Decodable {
    let name: String
}

let dataPublisher: AnyPublisher<Data, URLError> = URLSession.shared.dataTaskPublisher(for: URL(string: "https://example.com/user.json")!)
    .map(\.data)
    .eraseToAnyPublisher()

dataPublisher
    .tryMap { data -> User in
        try JSONDecoder().decode(User.self, from: data)
    }
    .sink(receiveCompletion: { print($0) },
          receiveValue: { user in print("User:", user.name) })
```

---

### 4. Использование с `mapError` для обработки ошибок

```swift
publisher
    .tryMap { value -> Int in
        guard value != 0 else { throw NSError(domain: "ZeroError", code: 1) }
        return 100 / value
    }
    .mapError { $0 as NSError } // преобразуем Error в конкретный тип
    .sink(receiveCompletion: { print($0) },
          receiveValue: { print($0) })
```

---

### 5. Цепочка операторов с `tryMap`

```swift
numbers.publisher
    .tryMap { 10 / $0 }
    .map { $0 * 2 }
    .filter { $0 > 5 }
    .sink(receiveCompletion: { print($0) },
          receiveValue: { print("Filtered Value:", $0) })
```
