## 1. Что такое Continuation

`Continuation` — это объект, который позволяет **превратить [[callback]]-based [[API]] (старые асинхронные функции) в [[async]]/[[await]]**.

- [[Swift]] Concurrency не может напрямую «ждать» старые callback-функции.
    
- Continuation позволяет **останавливать задачу до вызова callback**, а потом возобновлять выполнение с результатом.
    

Swift предоставляет два основных типа:

|Тип|Когда использовать|
|---|---|
|`CheckedContinuation`|Для функций, которые **не бросают ошибки**|
|`CheckedThrowingContinuation`|Для функций, которые **могут бросать ошибки**|

---

## 2. Основной синтаксис

```swift
func oldAPI(completion: @escaping (Result<String, Error>) -> Void) {
    // старый callback
}

func newAPI() async throws -> String {
    try await withCheckedThrowingContinuation { continuation in
        oldAPI { result in
            switch result {
            case .success(let value):
                continuation.resume(returning: value)
            case .failure(let error):
                continuation.resume(throwing: error)
            }
        }
    }
}
```

- `withCheckedContinuation` / `withCheckedThrowingContinuation` создаёт continuation и приостанавливает `async` функцию.
    
- Когда callback завершает работу — вызываем `continuation.resume(...)`, чтобы вернуть результат.
    

---

## 3. Примеры от простого к сложному

### Пример 1. Простой callback в async

```swift
func fetchDataOld(completion: @escaping (String) -> Void) {
    DispatchQueue.global().asyncAfter(deadline: .now() + 1) {
        completion("Данные старого API")
    }
}

func fetchDataNew() async -> String {
    await withCheckedContinuation { continuation in
        fetchDataOld { result in
            continuation.resume(returning: result)
        }
    }
}

// Использование
Task {
    let data = await fetchDataNew()
    print(data)
}
```

---

### Пример 2. Callback с ошибкой

```swift
enum NetworkError: Error {
    case failed
}

func fetchDataOld(completion: @escaping (Result<String, Error>) -> Void) {
    DispatchQueue.global().asyncAfter(deadline: .now() + 1) {
        let success = Bool.random()
        if success {
            completion(.success("Данные получены"))
        } else {
            completion(.failure(NetworkError.failed))
        }
    }
}

func fetchDataNew() async throws -> String {
    try await withCheckedThrowingContinuation { continuation in
        fetchDataOld { result in
            switch result {
            case .success(let value):
                continuation.resume(returning: value)
            case .failure(let error):
                continuation.resume(throwing: error)
            }
        }
    }
}

// Использование
Task {
    do {
        let data = try await fetchDataNew()
        print(data)
    } catch {
        print("Ошибка:", error)
    }
}
```

---

### Пример 3. Таймер через Continuation

```swift
func wait(seconds: UInt64) async {
    await withCheckedContinuation { continuation in
        DispatchQueue.global().asyncAfter(deadline: .now() + .seconds(Int(seconds))) {
            continuation.resume()
        }
    }
}

Task {
    print("Ждём 2 секунды…")
    await wait(seconds: 2)
    print("Готово!")
}
```

---

### Пример 4. Интеграция старого делегата

```swift
protocol DownloaderDelegate: AnyObject {
    func didFinishDownload(data: Data)
}

class Downloader {
    weak var delegate: DownloaderDelegate?
    
    func start() {
        DispatchQueue.global().asyncAfter(deadline: .now() + 1) {
            self.delegate?.didFinishDownload(data: Data([1,2,3]))
        }
    }
}

func downloadAsync(downloader: Downloader) async -> Data {
    await withCheckedContinuation { continuation in
        class Proxy: DownloaderDelegate {
            let cont: CheckedContinuation<Data, Never>
            init(cont: CheckedContinuation<Data, Never>) { self.cont = cont }
            func didFinishDownload(data: Data) { cont.resume(returning: data) }
        }
        
        let proxy = Proxy(cont: continuation)
        downloader.delegate = proxy
        downloader.start()
    }
}

// Использование
let downloader = Downloader()
Task {
    let data = await downloadAsync(downloader: downloader)
    print("Данные:", data)
}
```

---

### Пример 5. AsyncSequence через Continuation

```swift
func numbersStream() -> AsyncStream<Int> {
    AsyncStream { continuation in
        Task {
            for i in 1...5 {
                continuation.yield(i)
                try? await Task.sleep(nanoseconds: 300_000_000)
            }
            continuation.finish()
        }
    }
}

Task {
    for await n in numbersStream() {
        print(n)
    }
}
```

> Здесь Continuation скрыта внутри `AsyncStream`.

---

## 4. Особенности Continuation

1. **Ручной контроль**
    
    - Нужно вызывать `resume(returning:)` или `resume(throwing:)` **ровно один раз**.
        
2. **Проверка компилятора**
    
    - `withCheckedContinuation` проверяет, что `resume` вызывается один раз.
        
    - Есть `withUnsafeContinuation`, если нужен полный контроль (но без проверки).
        
3. **Идеально для интеграции старого API**
    
    - Делегаты, таймеры, [[URLSession]] с [[completion handler]], CoreBluetooth и т.д.
        

---

## 5. Итог

- Continuation = мост между **callback-based асинхронным кодом** и `async/await`.
    
- Основные функции: `withCheckedContinuation`, `withCheckedThrowingContinuation`.
    
- Используется для **таймеров, сетевых запросов, делегатов и старых библиотек**.
    
- Позволяет писать современный асинхронный код, не переписывая старые API.
    

---
