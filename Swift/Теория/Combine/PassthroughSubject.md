**`PassthroughSubject`** — тип **[[Publisher]]** в [[Combine]], который **не хранит значения**, но **передаёт их всем подписчикам**, как только они поступают.  
Идеален для **событийного потока данных**, где важно оповестить подписчиков о новых событиях, не сохраняя их.

---

## 🔹 Примеры кода

### 1. Создание и простая публикация

```swift
import Combine

let subject = PassthroughSubject<String, Never>()

let cancellable = subject.sink { value in
    print("Получено значение:", value)
}

subject.send("Hello")
subject.send("World")
```

---

### 2. Несколько подписчиков

```swift
let subject = PassthroughSubject<Int, Never>()

subject.sink { print("Подписчик 1:", $0) }.store(in: &cancellables)
subject.sink { print("Подписчик 2:", $0) }.store(in: &cancellables)

subject.send(10)
subject.send(20)
```

---

### 3. Обработка ошибок

```swift
enum MyError: Error { case failed }

let subject = PassthroughSubject<String, MyError>()

subject.sink(receiveCompletion: { completion in
    print("Завершение:", completion)
}, receiveValue: { value in
    print("Получено:", value)
})

subject.send("Data")
subject.send(completion: .failure(.failed))
```

---

### 4. Использование с UI событием

```swift
let buttonTap = PassthroughSubject<Void, Never>()

buttonTap.sink {
    print("Кнопка нажата")
}.store(in: &cancellables)

// Симуляция нажатия кнопки
buttonTap.send(())
```

---

### 5. Комбинирование с другими Publisher

```swift
let subject = PassthroughSubject<Int, Never>()
let publisher = [1, 2, 3].publisher

publisher
    .merge(with: subject)
    .sink { print("Merge:", $0) }
    .store(in: &cancellables)

subject.send(100)
```
