**URLSessionWebSocketDelegate** — это протокол в **Foundation**, который позволяет получать события и управлять **WebSocket-соединением**, созданным через `URLSessionWebSocketTask`.

В 2026 году это **единственный официальный** способ работать с **WebSocket** в нативных приложениях Apple (iOS 13+, macOS 10.15+, watchOS 6+, tvOS 13+, visionOS).

### Когда использовать именно URLSessionWebSocketDelegate

| Сценарий                                      | Почему именно этот delegate                      | Альтернатива (если не подходит) |
|-----------------------------------------------|--------------------------------------------------|---------------------------------|
| Нативный WebSocket без сторонних библиотек    | Полный контроль, интеграция с URLSession         | Starscream, Socket.IO, Alamofire WebSocket |
| Реал-тайм чат, уведомления, live-данные       | Низкая задержка, поддержка background            | Firebase Realtime / Firestore   |
| Фоновый WebSocket (работает после сворачивания) | Поддержка background-сессий                      | —                               |
| Кастомная аутентификация, пинг-понг, reconnect | `didOpen`, `didReceive`, `didClose`, `didReceive challenge` | Starscream / Socket.IO          |
| Интеграция с Combine / async/await            | Полный контроль над событиями                    | Combine + PassthroughSubject    |

### Самый популярный и рекомендуемый паттерн 2026 (actor + async/await + delegate)

```swift
import Foundation

actor WebSocketClient: NSObject, URLSessionWebSocketDelegate {
    
    private var session: URLSession!
    private var webSocketTask: URLSessionWebSocketTask?
    private var messageContinuation: AsyncThrowingStream<URLSessionWebSocketTask.Message, Error>.Continuation?
    
    override init() {
        super.init()
        
        let config = URLSessionConfiguration.default
        config.timeoutIntervalForRequest = 30
        config.timeoutIntervalForResource = 300
        
        session = URLSession(configuration: config, delegate: self, delegateQueue: nil)
    }
    
    func connect(to url: URL) async throws {
        webSocketTask = session.webSocketTask(with: url)
        webSocketTask?.resume()
        
        // Запускаем приём сообщений
        receiveMessages()
    }
    
    private func receiveMessages() {
        webSocketTask?.receive { [weak self] result in
            guard let self else { return }
            
            switch result {
            case .success(let message):
                Task { await self.handleMessage(message) }
                self.receiveMessages()  // рекурсивный вызов
                
            case .failure(let error):
                Task { await self.handleError(error) }
            }
        }
    }
    
    private func handleMessage(_ message: URLSessionWebSocketTask.Message) async {
        switch message {
        case .string(let text):
            print("Получено текстовое сообщение: \(text)")
            messageContinuation?.yield(.string(text))
            
        case .data(let data):
            print("Получено бинарное сообщение: \(data.count) байт")
            messageContinuation?.yield(.data(data))
            
        @unknown default:
            break
        }
    }
    
    private func handleError(_ error: Error) async {
        print("Ошибка WebSocket: \(error)")
        messageContinuation?.finish(throwing: error)
    }
    
    func send(_ message: URLSessionWebSocketTask.Message) async throws {
        try await withCheckedThrowingContinuation { continuation in
            webSocketTask?.send(message) { error in
                if let error {
                    continuation.resume(throwing: error)
                } else {
                    continuation.resume()
                }
            }
        }
    }
    
    func disconnect() {
        webSocketTask?.cancel(with: .goingAway, reason: nil)
        webSocketTask = nil
        messageContinuation?.finish()
    }
    
    // MARK: - URLSessionWebSocketDelegate
    
    nonisolated func urlSession(_ session: URLSession,
                                webSocketTask: URLSessionWebSocketTask,
                                didOpenWithProtocol protocol: String?) {
        Task { @MainActor in
            print("WebSocket открыт с протоколом: \(protocol ?? "none")")
        }
    }
    
    nonisolated func urlSession(_ session: URLSession,
                                webSocketTask: URLSessionWebSocketTask,
                                didCloseWith closeCode: URLSessionWebSocketTask.CloseCode,
                                reason: Data?) {
        Task { @MainActor in
            let reasonString = reason.flatMap { String(data: $0, encoding: .utf8) } ?? "нет причины"
            print("WebSocket закрыт: код \(closeCode.rawValue), причина: \(reasonString)")
        }
    }
    
    // Опционально: обработка challenge (аутентификация)
    nonisolated func urlSession(_ session: URLSession,
                                task: URLSessionTask,
                                didReceive challenge: URLAuthenticationChallenge,
                                completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {
        // Пример: доверие самоподписанному сертификату (только dev!)
        if challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust {
            completionHandler(.useCredential, URLCredential(trust: challenge.protectionSpace.serverTrust!))
        } else {
            completionHandler(.performDefaultHandling, nil)
        }
    }
}

// Использование
let client = WebSocketClient()

Task {
    do {
        try await client.connect(to: URL(string: "wss://echo.websocket.org")!)
        
        // Отправка
        try await client.send(.string("Привет, WebSocket!"))
        
        // Приём в цикле
        for try await message in client.messages {
            switch message {
            case .string(let text):
                print("Получено: \(text)")
            case .data(let data):
                print("Бинарные данные: \(data.count) байт")
            }
        }
    } catch {
        print("Ошибка: \(error)")
    }
}
```

### Лучшие практики URLSessionWebSocketDelegate в Swift 2026

- **Держи delegate в actor** — безопасно для concurrency  
- **nonisolated** — методы delegate должны быть `nonisolated` (вызываются не на main thread)  
- **AsyncStream** — идеально для приёма сообщений (`didReceive`)  
- **didOpenWithProtocol** — проверяй протокол (например, "chat")  
- **didCloseWith** — обрабатывай коды закрытия (1000 — нормальное, 1006 — ошибка соединения)  
- **Swift 6 strict concurrency** — delegate-методы **не** @MainActor → используй `Task { @MainActor in ... }` для UI  
- **Пинг-понг** — реализуй вручную (отправляй ping каждые 30 сек)  
- **Reconnect** — при `didCloseWith` или ошибке запускай reconnect-логику  
- **Документируйте** — пиши комментарий «URLSessionWebSocketDelegate — обработка WebSocket-соединения»

**Короткий девиз 2026**:
> «URLSessionWebSocketDelegate — это когда тебе нужен нативный, надёжный WebSocket с полным контролем: прогресс, reconnect, аутентификация и фон.  
> В 2026 году это **единственный** официальный способ от Apple для WebSocket.  
> Для простых чатов — Firebase / Supabase Realtime, для всего остального — URLSession + delegate.»

Удачи с реал-тайм связью в Swift! 🔌