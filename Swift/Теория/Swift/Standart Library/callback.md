## 1. Что такое callback

**Callback** — это **функция (или замыкание), которое передается как аргумент другой функции** и вызывается после завершения какой-то операции.

- Используется для уведомления о завершении работы или передачи результата
    
- Позволяет писать асинхронный код без блокировки потока
    

> Проще говоря: callback = «сообщение функции: когда закончишь, вызови меня».

---

## 2. Основные термины

| Термин                     | Описание                                                    |
| -------------------------- | ----------------------------------------------------------- |
| **Callback**               | Функция или closure, вызываемая после завершения операции   |
| **[[completion handler]]** | Синоним callback, часто используется в async API            |
| **Escaping closure**       | Callback, который сохраняется и вызывается позже (не сразу) |
| **Non-escaping closure**   | Callback, который вызывается внутри функции сразу           |

---

## 3. Основной синтаксис

```swift
func performTask(completion: () -> Void) {
    print("Task started")
    completion() // вызываем callback
}

performTask {
    print("Task finished")
}
```

- Output:
    

```
Task started
Task finished
```

- Здесь closure `{ print("Task finished") }` — это callback
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простейший callback

```swift
func greet(name: String, completion: () -> Void) {
    print("Hello, \(name)!")
    completion()
}

greet(name: "Alice") {
    print("Greeting finished")
}
```

- Callback вызывается **сразу после основной работы функции**
    

---

### Пример 2. Callback с параметром

```swift
func fetchData(completion: (String) -> Void) {
    let data = "Hello from server"
    completion(data)
}

fetchData { result in
    print("Received:", result)
}
```

- Callback может передавать результат
    

---

### Пример 3. [[Escaping callback]] (асинхронный)

```swift
func asyncTask(completion: @escaping (Int) -> Void) {
    DispatchQueue.global().async {
        let result = 42
        completion(result) // вызываем позже
    }
}

asyncTask { value in
    print("Result:", value)
}
```

- `@escaping` нужен, если closure сохраняется для вызова после выхода из функции
    

---

### Пример 4. Callback с ошибкой

```swift
enum NetworkError: Error { case failed }

func downloadData(completion: @escaping (Result<String, Error>) -> Void) {
    DispatchQueue.global().async {
        if Bool.random() {
            completion(.success("Data received"))
        } else {
            completion(.failure(NetworkError.failed))
        }
    }
}

downloadData { result in
    switch result {
    case .success(let data):
        print(data)
    case .failure(let error):
        print("Error:", error)
    }
}
```

- Используется [[Result]] для передачи ошибки или результата
    

---

### Пример 5. Несколько callbacks / chaining

```swift
func step1(completion: @escaping (Int) -> Void) {
    DispatchQueue.global().async { completion(1) }
}

func step2(input: Int, completion: @escaping (Int) -> Void) {
    DispatchQueue.global().async { completion(input + 1) }
}

func step3(input: Int, completion: @escaping (Int) -> Void) {
    DispatchQueue.global().async { completion(input * 2) }
}

step1 { result1 in
    step2(input: result1) { result2 in
        step3(input: result2) { result3 in
            print("Final result:", result3)
        }
    }
}
```

- Здесь видна **“callback hell”** — когда несколько асинхронных операций вызывают друг друга
    

---

## 5. Особенности callback

1. **Сразу vs поздно (escaping)**
    
    - Non-escaping: вызывается внутри функции
        
    - Escaping: может вызываться позже (например, [[async]])
        
2. **Передача результата**
    
    - Через параметры closure или `Result`
        
3. **Thread-safety**
    
    - Если callback вызывается на другом потоке, для UI нужно `DispatchQueue.main.async`
        
4. **Альтернатива современного async/await**
    
    - Callback использовался до [[Swift]] Concurrency
        
    - Сейчас часто заменяется `async` функциями
        

---

## 6. Итог

- **Callback** — функция/[[closure]], вызываемая после завершения операции
    
- Может передавать результат или ошибку
    
- Escaping closures нужны для асинхронных задач
    
- Часто ведёт к “callback hell” → теперь лучше использовать [[async]]/[[await]]
    

---
