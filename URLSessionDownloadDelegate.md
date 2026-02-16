**URLSessionDownloadDelegate** — это протокол в **Foundation**, который позволяет получать **детальные события** при выполнении задач типа `downloadTask` в `URLSession`.

Он используется, когда тебе нужно:

- скачивать файлы **с прогрессом** (progress tracking)  
- получать файл **по частям** или в виде временного URL  
- возобновлять прерванные загрузки (resume)  
- обрабатывать **редиректы**, **аутентификацию**, **фоновые загрузки**  
- работать с **background-сессиями** (фоновые загрузки после выхода из приложения)

В 2026 году это **единственный правильный способ** делать надёжные фоновые и прогрессивные загрузки файлов в iOS/macOS-приложениях.

### Когда использовать URLSessionDownloadDelegate (а не dataTask или data(from:))

| Сценарий                                      | Почему именно download delegate                  | Альтернатива (если не нужен) |
|-----------------------------------------------|--------------------------------------------------|------------------------------|
| Загрузка **больших файлов** (видео, PDF, образы, обновления) | Прогресс, resume, временный файл на диск         | `data(from:)` — только для маленьких ответов |
| **Фоновые загрузки** (работают после сворачивания/выхода приложения) | Обязателен для `URLSessionConfiguration.background` | Обычные сессии — не нужен |
| Показ **прогресс-бара** при скачивании        | `urlSession(_:downloadTask:didWriteData:totalBytesWritten:totalBytesExpectedToWrite:)` | Combine / async/await + progress |
| Возобновление прерванной загрузки             | `urlSession(_:downloadTask:didResumeAtOffset:expectedTotalBytes:)` | `resumeData` вручную |
| Кастомная обработка **редиректов** и **аутентификации** | `willPerformHTTPRedirection`, `didReceive challenge` | Автоматический редирект |
| Сохранение файла **на диск** без загрузки в память | `urlSession(_:downloadTask:didFinishDownloadingTo:)` даёт временный URL | `data(from:)` — загружает всё в память |

### Самый популярный и рекомендуемый паттерн 2026 (async/await + delegate)

```swift
import Foundation

actor FileDownloader: NSObject, URLSessionDownloadDelegate {
    
    private var session: URLSession!
    private var continuation: CheckedContinuation<(url: URL, response: URLResponse), Error>?
    private var progressContinuation: CheckedContinuation<Double, Never>?
    
    override init() {
        super.init()
        
        let config = URLSessionConfiguration.background(withIdentifier: "com.example.background.download")
        config.sessionSendsLaunchEvents = true
        config.isDiscretionary = true
        
        session = URLSession(configuration: config, delegate: self, delegateQueue: nil)
    }
    
    func downloadFile(from url: URL) async throws -> (localURL: URL, response: URLResponse) {
        return try await withCheckedThrowingContinuation { continuation in
            self.continuation = continuation
            
            let task = session.downloadTask(with: url)
            task.resume()
        }
    }
    
    func observeProgress() -> AsyncStream<Double> {
        AsyncStream { continuation in
            progressContinuation = continuation
        }
    }
    
    // MARK: - URLSessionDownloadDelegate
    
    nonisolated func urlSession(_ session: URLSession,
                                downloadTask: URLSessionDownloadTask,
                                didFinishDownloadingTo location: URL) {
        Task { @MainActor in
            guard let response = downloadTask.response else {
                continuation?.resume(throwing: DownloadError.noResponse)
                continuation = nil
                return
            }
            
            continuation?.resume(returning: (location, response))
            continuation = nil
        }
    }
    
    nonisolated func urlSession(_ session: URLSession,
                                downloadTask: URLSessionDownloadTask,
                                didWriteData bytesWritten: Int64,
                                totalBytesWritten: Int64,
                                totalBytesExpectedToWrite: Int64) {
        Task { @MainActor in
            guard totalBytesExpectedToWrite > 0 else { return }
            
            let progress = Double(totalBytesWritten) / Double(totalBytesExpectedToWrite)
            progressContinuation?.yield(progress)
        }
    }
    
    nonisolated func urlSession(_ session: URLSession,
                                task: URLSessionTask,
                                didCompleteWithError error: Error?) {
        Task { @MainActor in
            if let error {
                continuation?.resume(throwing: error)
                continuation = nil
            }
            progressContinuation?.finish()
        }
    }
    
    // Опционально: обработка resumeData при прерывании
    nonisolated func urlSession(_ session: URLSession,
                                task: URLSessionTask,
                                didReceive challenge: URLAuthenticationChallenge,
                                completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {
        // Обработка SSL, HTTP Basic и т.д.
        completionHandler(.useCredential, nil)
    }
}

// Использование
let downloader = FileDownloader()

Task {
    do {
        let (localURL, response) = try await downloader.downloadFile(from: URL(string: "https://example.com/largefile.zip")!)
        
        // Перемещаем файл в постоянное место
        let destination = FileManager.default.temporaryDirectory.appendingPathComponent("file.zip")
        try FileManager.default.moveItem(at: localURL, to: destination)
        
        print("Файл скачан в: \(destination)")
    } catch {
        print("Ошибка загрузки: \(error)")
    }
}

// Прогресс
for await progress in downloader.observeProgress() {
    print("Прогресс: \(progress * 100)%")
}
```

### Лучшие практики URLSessionDownloadDelegate в Swift 2026

- **Держи delegate в actor** — безопасно для concurrency  
- **nonisolated** — методы delegate должны быть `nonisolated`, т.к. вызываются не на main thread  
- **continuation** — используй `withCheckedThrowingContinuation` для async/await  
- **didFinishDownloadingTo** — здесь получаешь временный URL файла (перемещай его сразу!)  
- **didWriteData** — обновляй прогресс UI через `@MainActor`  
- **Swift 6 strict concurrency** — delegate-методы **не** @MainActor → используй `Task { @MainActor in ... }` для UI  
- **Фоновые сессии** — обязательно используй уникальный `identifier` и `sessionSendsLaunchEvents = true`  
- **Resume** — сохраняй `task.resumeData` при ошибке/прерывании  
- **Документируйте** — пиши комментарий «URLSessionDownloadDelegate — фоновая загрузка с прогрессом и resume»

**Короткий девиз 2026**:
> «URLSessionDownloadDelegate — это когда тебе нужно скачивать файлы с прогрессом, возобновлением, фоном и контролем над каждым этапом.  
> В 2026 году это **единственный** способ сделать настоящую надёжную фоновую загрузку.  
> Для простых случаев — `downloadTask` без delegate или Alamofire.»

Удачи с надёжными, прогрессивными и фоновыми загрузками в Swift! 📥