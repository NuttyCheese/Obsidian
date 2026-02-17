**Completion handler** — это **[[closure]]**, который передаётся в функцию и вызывается **после завершения операции**, чтобы передать результат или сигнализировать о завершении.

- Чаще всего используется в **асинхронных задачах**
    
- Позволяет функции **сообщить результат** без блокировки потока
    

> Проще говоря: completion handler = «callback, который срабатывает, когда задача закончилась».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**Completion handler**|Closure, переданный в функцию для уведомления о завершении|
|**Escaping closure**|Completion handler почти всегда escaping, т.к. вызывается позже|
|**Result type**|Стандартный тип `Result<Success, Error>` для передачи результата или ошибки|
|**Non-escaping closure**|Иногда вызывается сразу, но для async обычно escaping|

---

## 3. Основной синтаксис

```swift
func performTask(completion: () -> Void) {
    print("Task started")
    completion() // вызываем после выполнения
}

performTask {
    print("Task finished")
}
```

- Closure передаётся в функцию как аргумент
    
- Вызывается после выполнения операции
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простейший completion handler

```swift
func greet(name: String, completion: () -> Void) {
    print("Hello, \(name)!")
    completion()
}

greet(name: "Alice") {
    print("Greeting finished")
}
```

- Output:
    

```
Hello, Alice!
Greeting finished
```

---

### Пример 2. Completion handler с параметром

```swift
func fetchData(completion: (String) -> Void) {
    let data = "Server data"
    completion(data)
}

fetchData { result in
    print("Received:", result)
}
```

- Completion handler передаёт результат операции
    

---

### Пример 3. Escaping completion handler для асинхронного кода

```swift
func asyncTask(completion: @escaping (Int) -> Void) {
    DispatchQueue.global().async {
        let result = 42
        completion(result)
    }
}

asyncTask { value in
    print("Result:", value)
}
```

- `@escaping` нужен, чтобы closure можно было вызвать **после выхода из функции**
    

---

### Пример 4. Completion handler с [[Result]] для ошибок

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

- Используется `Result` для безопасной передачи ошибки или успеха
    

---

### Пример 5. Несколько completion handlers / chaining

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

- Демонстрирует **“callback hell”** при последовательных асинхронных операциях
    

---

## 5. Особенности completion handler

1. **Всегда closure** — callback-функция
    
2. **Escaping** — почти всегда вызывается позже, поэтому нужно `@escaping`
    
3. **Передача результата и ошибки** — через параметры или `Result`
    
4. **Удобство [[async]]/[[await]]** — современные функции заменяют completion handler на `async` функции
    

---

## 6. Итог

- **Completion handler** = closure, вызываемый после завершения операции
    
- Передаёт результат, ошибки или сигнал завершения
    
- Escaping closure для асинхронного вызова
    
- Используется в **network requests, animations, file operations, timers**
    

---
