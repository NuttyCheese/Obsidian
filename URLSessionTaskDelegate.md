**URLSessionTaskDelegate** — это протокол в **Foundation**, который позволяет получать **детальные события** для **любой** задачи (`URLSessionTask`) в `URLSession`: dataTask, downloadTask, uploadTask и streamTask.

Это **самый мощный** и **самый низкоуровневый** способ кастомизации сетевых операций в [[iOS]]/macOS-приложениях.

В 2026 году (Swift 6+, iOS 18+, macOS 15+) он используется, когда стандартные `data(from:)`, `downloadTask` или [[Alamofire]] недостаточно гибки.

### Когда использовать URLSessionTaskDelegate (а не просто data(from:) или downloadTask)

| Сценарий                                                                     | Почему нужен именно TaskDelegate                                                        | Альтернатива (если не нужен)          |
| ---------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | ------------------------------------- |
| **Прогресс загрузки/выгрузки** любого типа                                   | `urlSession(_:task:didSendBodyData:...)` и `urlSession(_:task:didReceiveChallenge:...)` | Простые запросы → `data(from:)`       |
| **Обработка [[HTTP]]-аутентификации** (Basic, Digest, NTLM, Kerberos, OAuth) | `urlSession(_:task:didReceive:challenge:completionHandler:)`                            | Bearer-токен в заголовках             |
| **Кастомные редиректы**                                                      | `urlSession(_:task:willPerformHTTPRedirection:...)`                                     | Автоматический редирект               |
| **Управление кэшированием вручную**                                          | `urlSession(_:task:willCacheResponse:completionHandler:)`                               | Автоматический кэш                    |
| **Сбор метрик** (время, размер, тип соединения)                              | `urlSession(_:task:didFinishCollecting:)`                                               | Instruments / Alamofire metrics       |
| **Обработка ошибок сертификатов / SSL**                                      | `didReceive challenge`                                                                  | Доверенные сертификаты в конфигурации |
| **Фоновые сессии** (background upload/download)                              | Обязателен для `URLSessionConfiguration.background`                                     | Обычные сессии — не нужен             |
| **Логирование всех событий** (headers, timing, redirects)                    | Полный контроль над жизненным циклом задачи                                             | Alamofire interceptors                |

### Самый популярный и рекомендуемый паттерн 2026 ([[async]]/[[await]] + TaskDelegate)

```swift
import Foundation

actor AdvancedNetworkClient: NSObject, URLSessionTaskDelegate {
    
    private var session: URLSession!
    private var taskContinuations: [URLSessionTask: CheckedContinuation<(data: Data?, response: URLResponse?, error: Error?), Error>] = [:]
    private var progressContinuations: [URLSessionTask: CheckedContinuation<Double, Never>] = [:]
    
    override init() {
        super.init()
        
        let config = URLSessionConfiguration.default
        config.timeoutIntervalForRequest = 30
        config.waitsForConnectivity = true
        config.httpAdditionalHeaders = ["User-Agent": "MyApp/1.0"]
        
        session = URLSession(configuration: config, delegate: self, delegateQueue: nil)
    }
    
    func performRequest(_ request: URLRequest) async throws -> (data: Data, response: HTTPURLResponse) {
        try await withCheckedThrowingContinuation { continuation in
            let task = session.dataTask(with: request)
            taskContinuations[task] = continuation
            task.resume()
        }
    }
    
    func observeProgress(for task: URLSessionTask) -> AsyncStream<Double> {
        AsyncStream { continuation in
            progressContinuations[task] = continuation
        }
    }
    
    // MARK: - URLSessionTaskDelegate
    
    nonisolated func urlSession(_ session: URLSession,
                                task: URLSessionTask,
                                didCompleteWithError error: Error?) {
        Task { @MainActor in
            if let continuation = taskContinuations.removeValue(forKey: task) {
                if let error {
                    continuation.resume(throwing: error)
                } else if let dataTask = task as? URLSessionDataTask,
                          let response = dataTask.response,
                          let data = /* собранные данные из didReceive data */ {
                    continuation.resume(returning: (data, response, nil))
                }
            }
            progressContinuations.removeValue(forKey: task)?.finish()
        }
    }
    
    nonisolated func urlSession(_ session: URLSession,
                                task: URLSessionTask,
                                didSendBodyData bytesSent: Int64,
                                totalBytesSent: Int64,
                                totalBytesExpectedToSend: Int64) {
        Task { @MainActor in
            guard totalBytesExpectedToSend > 0,
                  let continuation = progressContinuations[task] else { return }
            
            let progress = Double(totalBytesSent) / Double(totalBytesExpectedToSend)
            continuation.yield(progress)
        }
    }
    
    nonisolated func urlSession(_ session: URLSession,
                                task: URLSessionTask,
                                willPerformHTTPRedirection response: HTTPURLResponse,
                                newRequest request: URLRequest?,
                                completionHandler: @escaping (URLRequest?) -> Void) {
        // Можно изменить request или отменить редирект
        completionHandler(request)
    }
    
    nonisolated func urlSession(_ session: URLSession,
                                task: URLSessionTask,
                                didReceive challenge: URLAuthenticationChallenge,
                                completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {
        // Обработка SSL, HTTP Basic, NTLM и т.д.
        if challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust {
            // Пример: доверие самоподписанному сертификату (только для dev!)
            completionHandler(.useCredential, URLCredential(trust: challenge.protectionSpace.serverTrust!))
        } else {
            completionHandler(.performDefaultHandling, nil)
        }
    }
    
    nonisolated func urlSession(_ session: URLSession,
                                task: URLSessionTask,
                                didFinishCollecting metrics: URLSessionTaskMetrics) {
        // Логирование времени, типа соединения, размера и т.д.
        Task { @MainActor in
            print("Метрики запроса: \(metrics)")
        }
    }
}
```

### Лучшие практики URLSessionTaskDelegate в Swift 2026

- **Держи delegate в actor** — безопасно для concurrency  
- **nonisolated** — методы delegate должны быть `nonisolated`, т.к. вызываются не на main thread  
- **continuation** — используй `withCheckedThrowingContinuation` для async/await  
- **didCompleteWithError** — всегда завершай continuation здесь  
- **didSendBodyData** / **didWriteData** — обновляй прогресс UI через `@MainActor`  
- **Swift 6 strict concurrency** — delegate-методы **не** @MainActor → используй `Task { @MainActor in ... }` для UI  
- **Фоновые сессии** — обязательно используй `URLSessionConfiguration.background` + уникальный identifier  
- **Метрики** — собирай в `didFinishCollecting metrics` для аналитики производительности  
- **Документируйте** — пиши комментарий «URLSessionTaskDelegate — полный контроль над задачей»

**Короткий девиз 2026**:
> «URLSessionTaskDelegate — это когда тебе нужен полный контроль над жизненным циклом любой сетевой задачи: прогресс, редиректы, аутентификация, метрики и фон.  
> В 2026 году это **единственный** способ сделать по-настоящему кастомную и надёжную сетевую логику.  
> Для простых запросов — `data(from:)` или Alamofire.»
