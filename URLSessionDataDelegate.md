**URLSessionDataDelegate** — это протокол в **Foundation**, который позволяет получать **детальные события** при выполнении задач типа `dataTask` в `URLSession`.

Он используется, когда тебе нужно:

- получать данные **по частям** (progressive loading)  
- обрабатывать **заголовки ответа** до завершения загрузки  
- отслеживать **прогресс** загрузки  
- реагировать на **редиректы**, **аутентификацию**, **кэширование**  
- работать с **фоновыми сессиями** (background download/upload)

В 2026 году (Swift 6+, iOS 18+, macOS 15+) это **один из самых мощных** способов кастомизировать сетевые запросы, особенно когда `URLSession.shared.data(from:)` недостаточно гибок.

### Когда использовать URLSessionDataDelegate (а не просто data(from:))

| Сценарий                                      | Почему нужен именно delegate                     | Альтернатива (если не нужен) |
|-----------------------------------------------|--------------------------------------------------|------------------------------|
| Показывать **прогресс-бар** при загрузке файла/изображения | `urlSession(_:task:didSendBodyData:totalBytesSent:totalBytesExpectedToSend:)` | Combine / async/await + progress publisher |
| Получать данные **по частям** (streaming)     | `urlSession(_:dataTask:didReceive:)` вызывается многократно | Простые запросы → `data(from:)` |
| Обрабатывать **редиректы** вручную            | `urlSession(_:task:willPerformHTTPRedirection:)` | Автоматический редирект по умолчанию |
| Кастомная **аутентификация** (HTTP Basic, Digest, NTLM, OAuth) | `urlSession(_:task:didReceive:challenge:completionHandler:)` | Bearer-токен в заголовках |
| Работа с **фоновыми сессиями** (background download/upload) | Обязателен для `URLSessionConfiguration.background` | Обычные сессии → не нужен |
| Управление **кэшированием** вручную           | `urlSession(_:dataTask:willCacheResponse:completionHandler:)` | Автоматический кэш |
| Логирование **всех событий** (headers, metrics) | `urlSession(_:task:didFinishCollecting:)` | Alamofire / URLSessionTaskDelegate |

### Самый популярный и рекомендуемый паттерн 2026 (async/await + delegate)

```swift
import Foundation

actor ImageDownloader: NSObject, URLSessionDataDelegate {
    
    private var session: URLSession!
    private var continuation: CheckedContinuation<Data, Error>?
    private var receivedData = Data()
    
    override init() {
        super.init()
        
        let config = URLSessionConfiguration.default
        config.timeoutIntervalForRequest = 30
        config.waitsForConnectivity = true
        
        session = URLSession(configuration: config, delegate: self, delegateQueue: nil)
    }
    
    func downloadImage(from url: URL) async throws -> Data {
        receivedData = Data()
        
        return try await withCheckedThrowingContinuation { continuation in
            self.continuation = continuation
            
            let task = session.dataTask(with: url)
            task.resume()
        }
    }
    
    // MARK: - URLSessionDataDelegate
    
    nonisolated func urlSession(_ session: URLSession,
                                dataTask: URLSessionDataTask,
                                didReceive data: Data) {
        // Данные приходят по частям
        Task { @MainActor in
            self.receivedData.append(data)
            
            // Можно обновлять UI-прогресс
            if let total = dataTask.countOfBytesExpectedToReceive,
               total > 0 {
                let progress = Double(dataTask.countOfBytesReceived) / Double(total)
                print("Прогресс: \(progress * 100)%")
            }
        }
    }
    
    nonisolated func urlSession(_ session: URLSession,
                                task: URLSessionTask,
                                didCompleteWithError error: Error?) {
        Task { @MainActor in
            if let error {
                self.continuation?.resume(throwing: error)
            } else {
                self.continuation?.resume(returning: self.receivedData)
            }
            self.continuation = nil
            self.receivedData = Data()
        }
    }
    
    // Опционально: обработка редиректов
    nonisolated func urlSession(_ session: URLSession,
                                task: URLSessionTask,
                                willPerformHTTPRedirection response: HTTPURLResponse,
                                newRequest request: URLRequest,
                                completionHandler: @escaping (URLRequest?) -> Void) {
        // Можно изменить request или отменить редирект
        completionHandler(request)
    }
}

// Использование
let downloader = ImageDownloader()

Task {
    do {
        let imageData = try await downloader.downloadImage(from: URL(string: "https://example.com/image.jpg")!)
        // обработка изображения
    } catch {
        print("Ошибка загрузки: \(error)")
    }
}
```

### Лучшие практики URLSessionDataDelegate в Swift 2026

- **Держи delegate в actor** — безопасно для concurrency  
- **nonisolated** — методы delegate должны быть `nonisolated`, т.к. вызываются не на main thread  
- **continuation** — используй `withCheckedThrowingContinuation` для async/await  
- **receivedData.append** — собирай данные по частям в `didReceive data:`  
- **didCompleteWithError** — всегда завершай continuation здесь  
- **Swift 6 strict concurrency** — delegate-методы **не** @MainActor → используй `Task { @MainActor in ... }` для UI-обновлений  
- **Прогресс** — обновляй через `didSendBodyData` / `didReceive data`  
- **Документируйте** — пиши комментарий «URLSessionDataDelegate — потоковая загрузка с прогрессом»

**Короткий девиз 2026**:
> «URLSessionDataDelegate — это когда тебе нужно получать данные по частям, показывать прогресс, обрабатывать редиректы или работать с фоновыми сессиями.  
> В 2026 году это **единственный** способ сделать мощную кастомную сетевую загрузку.  
> Для простых запросов — `data(from:)` или Alamofire.»

Удачи с потоковой, прогрессивной и кастомной загрузкой в Swift! 📡