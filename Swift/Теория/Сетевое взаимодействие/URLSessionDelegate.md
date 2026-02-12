#network #Swift 
**`URLSessionDelegate`** — это основной протокол, который позволяет **получать уведомления и управлять поведением** [[URLSession]] на низком уровне.

Он используется, когда стандартных [[async]]/[[await]] или completion-методов недостаточно, и нужно:

- обрабатывать аутентификацию (сервер/клиент)  
- управлять перенаправлениями  
- отслеживать прогресс загрузки/выгрузки  
- реагировать на ошибки сессии  
- работать с фоновыми сессиями  
- реализовывать кастомную логику (certificate pinning, retry и т.д.)

### 1. Иерархия протоколов-делегатов URLSession

| Протокол                        | Наследуется от             | Основная задача                                       | Когда нужен (2026)                | Частота использования |
| ------------------------------- | -------------------------- | ----------------------------------------------------- | --------------------------------- | --------------------- |
| [[URLSessionDelegate]]          | [[NSObjectProtocol]]       | Общие события сессии (инвалидация, background)        | Фоновые сессии, глобальные ошибки | ★★★☆☆                 |
| [[URLSessionTaskDelegate]]      | [[URLSessionDelegate]]     | События отдельных задач (редирект, прогресс)          | Прогресс, редирект, retry         | ★★★★☆                 |
| [[URLSessionDataDelegate]]      | [[URLSessionTaskDelegate]] | Потоковая загрузка данных (data task)                 | Большой [[JSON]] потоково         | ★★☆☆☆                 |
| [[URLSessionDownloadDelegate]]  | [[URLSessionTaskDelegate]] | Завершение скачивания файла (download task)           | Фоновые загрузки файлов           | ★★★☆☆                 |
| [[URLSessionWebSocketDelegate]] | [[URLSessionTaskDelegate]] | События [[WebSocket]] (открытие, закрытие, сообщение) | WebSocket-соединения              | ★★☆☆☆                 |

**Правило 2026 года**:  
В 95% случаев достаточно [[async]]/[[await]] без делегатов.  
Делегаты нужны только для **фоновых загрузок**, **прогресса**, **certificate pinning**, **редиректов** и **WebSocket**.

### 2. Ключевые методы URLSessionDelegate

| Метод                                                                  | Когда вызывается                                            | Что нужно делать                         |
| ---------------------------------------------------------------------- | ----------------------------------------------------------- | ---------------------------------------- |
| `urlSession(_:didBecomeInvalidWithError:)`                             | Сессия стала недействительной (ошибка, invalidate)          | Логировать, пересоздать сессию           |
| `urlSession(_:didReceive challenge:completionHandler:)`                | Сервер требует аутентификацию ([[HTTPS]], Basic Auth, NTLM) | Certificate pinning, обработка challenge |
| `urlSessionDidFinishEvents(forBackgroundURLSession:)`                  | Все фоновые задачи завершены                                | Обработать завершение фона               |
| `urlSession(_:task:didCompleteWithError:)` (из URLSessionTaskDelegate) | Задача завершена (успех или ошибка)                         | Логировать ошибки                        |

### 3. Полные примеры кода

#### Пример 1 — Certificate Pinning (самый частый продвинутый случай)

```swift
class SecureSessionDelegate: NSObject, URLSessionDelegate {
    func urlSession(_ session: URLSession,
                    didReceive challenge: URLAuthenticationChallenge,
                    completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {
        
        guard challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust,
              let serverTrust = challenge.protectionSpace.serverTrust else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }
        
        // Здесь можно проверить публичный ключ сервера (pinning)
        // Пример: сравнить с известным сертификатом
        
        let credential = URLCredential(trust: serverTrust)
        completionHandler(.useCredential, credential)
    }
}

// Создание сессии
let config = URLSessionConfiguration.default
let secureSession = URLSession(configuration: config,
                               delegate: SecureSessionDelegate(),
                               delegateQueue: nil)
```

#### Пример 2 — Прогресс загрузки (URLSessionTaskDelegate + downloadTask)

```swift
class DownloadManager: NSObject, URLSessionDownloadDelegate, URLSessionTaskDelegate {
    var progressHandler: ((Double) -> Void)?
    var completionHandler: ((URL?, Error?) -> Void)?
    
    func urlSession(_ session: URLSession,
                    downloadTask: URLSessionDownloadTask,
                    didWriteData bytesWritten: Int64,
                    totalBytesWritten: Int64,
                    totalBytesExpectedToWrite: Int64) {
        let progress = Double(totalBytesWritten) / Double(totalBytesExpectedToWrite)
        progressHandler?(progress)
    }
    
    func urlSession(_ session: URLSession,
                    downloadTask: URLSessionDownloadTask,
                    didFinishDownloadingTo location: URL) {
        completionHandler?(location, nil)
    }
    
    func urlSession(_ session: URLSession,
                    task: URLSessionTask,
                    didCompleteWithError error: Error?) {
        if let error = error {
            completionHandler?(nil, error)
        }
    }
}

// Использование
let config = URLSessionConfiguration.default
let delegate = DownloadManager()
let session = URLSession(configuration: config, delegate: delegate, delegateQueue: nil)

let url = URL(string: "https://example.com/largefile.zip")!
let task = session.downloadTask(with: url)
task.resume()

delegate.progressHandler = { progress in
    print("Прогресс: \(Int(progress * 100))%")
}
```

#### Пример 3 — Обработка фоновой сессии (background completion)

```swift
class BackgroundDownloader: NSObject, URLSessionDownloadDelegate {
    var backgroundCompletionHandler: (() -> Void)?
    
    func urlSessionDidFinishEvents(forBackgroundURLSession session: URLSession) {
        DispatchQueue.main.async {
            self.backgroundCompletionHandler?()
            self.backgroundCompletionHandler = nil
        }
    }
    
    func urlSession(_ session: URLSession,
                    downloadTask: URLSessionDownloadTask,
                    didFinishDownloadingTo location: URL) {
        // Переместить файл
        let destination = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
            .appendingPathComponent(location.lastPathComponent)
        try? FileManager.default.moveItem(at: location, to: destination)
    }
}

// В AppDelegate / SceneDelegate
func application(_ application: UIApplication,
                 handleEventsForBackgroundURLSession identifier: String,
                 completionHandler: @escaping () -> Void) {
    backgroundDownloader.backgroundCompletionHandler = completionHandler
}
```

#### Пример 4 — Отмена редиректов

```swift
class RedirectController: NSObject, URLSessionTaskDelegate {
    func urlSession(_ session: URLSession,
                    task: URLSessionTask,
                    willPerformHTTPRedirection response: HTTPURLResponse,
                    newRequest request: URLRequest,
                    completionHandler: @escaping (URLRequest?) -> Void) {
        // Отменяем все редиректы
        completionHandler(nil)
        print("Редирект отменён с", response.url?.absoluteString ?? "")
    }
}
```

### 4. Таблица: ключевые методы делегатов

| Протокол                     | Метод                        | Когда вызывается                    | Что нужно возвращать / делать                          |
| ---------------------------- | ---------------------------- | ----------------------------------- | ------------------------------------------------------ |
| `URLSessionDelegate`         | `didBecomeInvalidWithError`  | Сессия сломалась                    | Логировать, пересоздать сессию                         |
| `URLSessionDelegate`         | `didReceive challenge`       | Сервер требует аутентификацию       | `.useCredential`, `.cancel`, `.performDefaultHandling` |
| `URLSessionTaskDelegate`     | `willPerformHTTPRedirection` | Происходит HTTP-редирект            | Новый [[URLRequest]]? или [[nil]] (отмена)             |
| `URLSessionTaskDelegate`     | `didCompleteWithError`       | Задача завершена (успех или ошибка) | Обработать ошибку                                      |
| `URLSessionDownloadDelegate` | `didFinishDownloadingTo`     | Файл скачан в временную локацию     | Переместить файл                                       |
| `URLSessionDownloadDelegate` | `didWriteData`               | Прогресс скачивания                 | Обновить UI                                            |
| `URLSessionDataDelegate`     | `didReceive data`            | Получена новая порция данных        | Собирать данные вручную                                |

### 5. Лучшие практики 2026 года

- В 95% случаев используй **async/await** без делегатов  
- Делегаты нужны только для:  
  - certificate pinning  
  - прогресса загрузки/выгрузки  
  - фоновых сессий  
  - кастомных редиректов  
  - WebSocket  
- Используй **один делегат** на сессию (не создавай на каждую задачу)  
- Всегда держи сильную ссылку на делегат (или weak, но с осторожностью)  
- Для прогресса — обновляй UI на main queue  
- Для background — сохраняй completionHandler и вызывай его после обработки  
- Логируй все ошибки и статусы для отладки

**Короткий девиз**:
> «URLSessionDelegate — это низкоуровневый контроль.  
> Используй его только когда async/await недостаточно.  
> В 2026 году — 90% сетевых задач решаются без делегатов.»
