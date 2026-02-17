**AnyCancellable** — это тип из [[Combine]], представляющий собой объект, который управляет жизненным циклом подписки на [[Publisher]].  
Когда экземпляр `AnyCancellable` деинициализируется или вызывается метод `cancel()`, соответствующая подписка прекращается.

Проще говоря: **AnyCancellable — это «диспозер» Combine-подписок**, аналог `DisposeBag` из [[RxSwift]], но без мешка — подписки хранятся в коллекции.

---

### **К какому стеку относится**

- **Combine Framework**
    
- Основа: **[[Foundation]]**
    
- Используется в любом UI-слое: [[UIKit]] / [[SwiftUI]]
    

---

## **1. Простая подписка и отмена**

```swift
import Combine

let publisher = Just("Hello")
let cancellable = publisher.sink { value in
    print(value)
}

cancellable.cancel() // отменяем подписку вручную
```

**Комментарий:** вручную прекращаем подписку.

---

## **2. Хранение AnyCancellable в Set**

```swift
var cancellables = Set<AnyCancellable>()

Just(10)
    .sink { value in
        print("Получено:", value)
    }
    .store(in: &cancellables) // подписка сохранена
```

**Комментарий:** стандартный способ хранения подписок.

---

## **3. Подписка на NotificationCenter Publisher**

```swift
var cancellables = Set<AnyCancellable>()

NotificationCenter.default.publisher(for: UIApplication.didBecomeActiveNotification)
    .sink { _ in
        print("Приложение снова активно")
    }
    .store(in: &cancellables)
```

**Комментарий:** реальный пример для UIKit.

---

## **4. Подписка на @Published свойство**

```swift
class ViewModel {
    @Published var text: String = ""
    var cancellables = Set<AnyCancellable>()

    init() {
        $text
            .sink { value in
                print("Текст изменён:", value)
            }
            .store(in: &cancellables)
    }
}

let vm = ViewModel()
vm.text = "Привет"
```

**Комментарий:** Combine автоматически генерирует publisher для `@Published`.

---

## **5. Подписка с обработкой ошибок**

```swift
enum TestError: Error { case bad }

let publisher = Fail<String, TestError>(error: .bad)

var cancellables = Set<AnyCancellable>()

publisher
    .sink(receiveCompletion: { completion in
        print("Завершение:", completion)
    }, receiveValue: { value in
        print("Значение:", value)
    })
    .store(in: &cancellables)
```

**Комментарий:** `AnyCancellable` хранит подписку до завершения.

---

## **6. Применение в сетевом слое (URLSession + Combine)**

```swift
import Combine
import Foundation

class NetworkService {
    private var cancellables = Set<AnyCancellable>()

    func loadUsers() {
        URLSession.shared.dataTaskPublisher(for: URL(string: "https://api.example.com/users")!)
            .map(\.data)
            .decode(type: [User].self, decoder: JSONDecoder())
            .sink(receiveCompletion: { completion in
                print("Completion:", completion)
            }, receiveValue: { users in
                print("Loaded users:", users)
            })
            .store(in: &cancellables)
    }
}

let service = NetworkService()
service.loadUsers()
```

**Комментарий:** типичная архитектура — сервис держит подписки внутри себя.
