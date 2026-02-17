`combineLatest` — оператор в [[Combine]] и [[RxSwift]], который объединяет несколько `Publisher` в один.  
Он выдаёт **новое значение каждый раз, когда один из источников публикует новое значение**, при этом берёт **последние значения всех Publisher**.  
Используется для синхронизации нескольких потоков данных.

---

## 🔹 Примеры кода

### 1. Простое объединение двух `Just` [[Publisher]]

```swift
import Combine

let publisher1 = Just(1)
let publisher2 = Just(10)

publisher1.combineLatest(publisher2)
    .sink { value1, value2 in
        print("combineLatest:", value1, value2)
    }
```

---

### 2. Объединение двух `PassthroughSubject`

```swift
let subject1 = PassthroughSubject<String, Never>()
let subject2 = PassthroughSubject<Int, Never>()

subject1.combineLatest(subject2)
    .sink { str, num in
        print("combineLatest:", str, num)
    }

subject1.send("Hello")
subject2.send(100)
subject1.send("World")
```

---

### 3. Использование с `@Published` свойствами

```swift
class ViewModel {
    @Published var firstName = ""
    @Published var lastName = ""
    
    var cancellables = Set<AnyCancellable>()
    
    init() {
        $firstName.combineLatest($lastName)
            .sink { first, last in
                print("Полное имя:", first, last)
            }
            .store(in: &cancellables)
    }
}

let vm = ViewModel()
vm.firstName = "Иван"
vm.lastName = "Иванов"
```

---

### 4. Объединение трёх Publisher

```swift
let a = PassthroughSubject<Int, Never>()
let b = PassthroughSubject<Int, Never>()
let c = PassthroughSubject<Int, Never>()

a.combineLatest(b, c)
    .sink { x, y, z in
        print("combineLatest 3:", x, y, z)
    }

a.send(1)
b.send(2)
c.send(3)
```

---

### 5. Применение в сетевых запросах (симуляция)

```swift
func fetchUser() -> AnyPublisher<String, Never> {
    Just("User").eraseToAnyPublisher()
}

func fetchSettings() -> AnyPublisher<String, Never> {
    Just("Settings").eraseToAnyPublisher()
}

fetchUser().combineLatest(fetchSettings())
    .sink { user, settings in
        print("Данные готовы:", user, settings)
    }
```

---

### 6. С фильтрацией значений после `combineLatest`

```swift
let temperature = PassthroughSubject<Int, Never>()
let humidity = PassthroughSubject<Int, Never>()

temperature.combineLatest(humidity)
    .filter { temp, hum in temp > 25 && hum < 50 }
    .sink { temp, hum in
        print("Жарко и сухо:", temp, hum)
    }

temperature.send(26)
humidity.send(45)
temperature.send(24)
humidity.send(40)
```
