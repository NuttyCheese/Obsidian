`Cancellable` — это протокол в Combine, описывающий объект, который может отменить выполняемую работу (обычно — подписку на [[Publisher]]).  
Любой тип, соответствующий `Cancellable`, обязан иметь метод `cancel()`.  
`AnyCancellable` — одна из основных реализаций протокола.

---

## 🔹 Примеры кода

### 1. Простая реализация `Cancellable`

```swift
import Combine

class SimpleCancel: Cancellable {
    func cancel() {
        print("Отмена операции")
    }
}

let c = SimpleCancel()
c.cancel()
```

---

### 2. Использование стандартного `AnyCancellable`

```swift
import Combine

let cancellable: Cancellable = Just("Hello")
    .sink { print($0) }

cancellable.cancel() // отменяем подписку
```

---

### 3. Кастомный объект с логикой отмены

```swift
class NetworkTask: Cancellable {
    private var isCancelled = false

    func cancel() {
        isCancelled = true
        print("Запрос отменён")
    }
}

let task = NetworkTask()
task.cancel()
```

---

### 4. Хранение нескольких `Cancellable` в коллекции

```swift
var cancellables: [Cancellable] = []

let c1 = AnyCancellable { print("Отмена 1") }
let c2 = AnyCancellable { print("Отмена 2") }

cancellables.append(c1)
cancellables.append(c2)

cancellables.forEach { $0.cancel() }
```

---

### 5. Использование Cancellable внутри класса

```swift
class ViewModel {
    var subscription: Cancellable?

    func start() {
        subscription = Timer.publish(every: 1, on: .main, in: .default)
            .autoconnect()
            .sink { _ in print("Тик") }
    }

    func stop() {
        subscription?.cancel()
    }
}

let vm = ViewModel()
vm.start()
vm.stop()
```

---

### 6. Сложный пример с кастомным Publisher

```swift
struct CounterPublisher: Publisher {
    typealias Output = Int
    typealias Failure = Never

    func receive<S>(subscriber: S) where S : Subscriber, Never == S.Failure, Int == S.Input {
        let subscription = CounterSubscription(subscriber: subscriber)
        subscriber.receive(subscription: subscription)
    }
}

final class CounterSubscription<S: Subscriber>: Cancellable, Subscription where S.Input == Int {
    private var subscriber: S?
    private var value = 0

    init(subscriber: S) {
        self.subscriber = subscriber
    }

    func request(_ demand: Subscribers.Demand) {
        guard demand > .none, let subscriber else { return }
        _ = subscriber.receive(value)
    }

    func cancel() {
        subscriber = nil
    }
}
```

---
