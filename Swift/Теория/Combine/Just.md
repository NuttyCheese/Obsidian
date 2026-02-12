#combine #reactive #Swift 
## 📘 Определение

**`Just`** — **[[Publisher]]** в [[Combine]], который выпускает **одно значение и завершает поток**.  
Используется для быстрого создания Publisher с одним элементом, часто для тестирования, проброса констант или интеграции с другими операторами Combine.

---

## 🔹 Примеры кода

### 1. Простейший `Just`

```swift
import Combine

let publisher = Just("Hello Combine")

let cancellable = publisher.sink { value in
    print("Получено значение:", value)
}
```

---

### 2. `Just` с числом

```swift
let numberPublisher = Just(42)

numberPublisher.sink { print("Число:", $0) }
```

---

### 3. `Just` с указанием типа ошибки

```swift
enum MyError: Error {
    case failed
}

let publisher = Just("Success")
    .setFailureType(to: MyError.self) // Just по умолчанию Never
```

---

### 4. Комбинирование с другими операторами

```swift
Just(5)
    .map { $0 * 2 }
    .sink { print("Результат:", $0) } // 10
```

---

### 5. Использование в [[flatMap]]

```swift
Just("Swift")
    .flatMap { value in
        Just(value.uppercased())
    }
    .sink { print($0) } // SWIFT
```

---

### 6. Тестирование асинхронного кода

```swift
func fetchValue() -> AnyPublisher<String, Never> {
    Just("Fetched Value").eraseToAnyPublisher()
}

let cancellable = fetchValue().sink { print($0) }
```
