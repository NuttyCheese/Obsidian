**Future** — это тип в [[Combine]], представляющий **одноразовый результат асинхронной операции**, который может завершиться **успешно (`Output`)** или с **ошибкой (`Failure`)**.  
Позволяет интегрировать одноразовые асинхронные действия в реактивный поток, поддерживая [[Chaining]] и composition с другими [[Publisher]].

---

## 🔹 Примеры кода

### 1. Создание простого Future

```swift
import Combine
import Foundation

let future = Future<String, Never> { promise in
    promise(.success("Hello Future"))
}

let cancellable = future.sink { value in
    print("Получено значение:", value)
}
```

---

### 2. Future с задержкой

```swift
let delayedFuture = Future<String, Never> { promise in
    DispatchQueue.global().asyncAfter(deadline: .now() + 2) {
        promise(.success("Задержка завершена"))
    }
}

delayedFuture.sink { print($0) }
```

---

### 3. Future с возможной ошибкой

```swift
enum NetworkError: Error {
    case failed
}

let networkFuture = Future<String, NetworkError> { promise in
    let success = Bool.random()
    if success {
        promise(.success("Данные загружены"))
    } else {
        promise(.failure(.failed))
    }
}

networkFuture.sink { completion in
    switch completion {
    case .finished:
        print("Операция завершена")
    case .failure(let error):
        print("Ошибка:", error)
    }
} receiveValue: { value in
    print("Значение:", value)
}
```

---

### 4. Использование Future внутри функции

```swift
func fetchData() -> Future<String, Never> {
    Future { promise in
        DispatchQueue.global().asyncAfter(deadline: .now() + 1) {
            promise(.success("Данные из функции"))
        }
    }
}

let cancellable = fetchData().sink { print($0) }
```

---

### 5. Комбинирование Future с другими [[Publisher]]

```swift
let future1 = Future<Int, Never> { $0(.success(5)) }
let future2 = Future<Int, Never> { $0(.success(10)) }

future1.combineLatest(future2)
    .sink { value1, value2 in
        print("Сумма:", value1 + value2)
    }
```
