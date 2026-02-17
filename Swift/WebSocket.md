**WebSocket** — это протокол (RFC 6455), обеспечивающий **двустороннюю постоянную связь** между клиентом и сервером поверх TCP.

В отличие от [[HTTP]] (запрос-ответ), WebSocket после handshake остаётся открытым — обе стороны могут **отправлять сообщения в любой момент**.

### 1. Когда использовать WebSocket в [[iOS]] (2026)

| Сценарий                                 | Почему WebSocket лучше [[HTTP]] polling/long polling | Рекомендуемый инструмент                 |
| ---------------------------------------- | ---------------------------------------------------- | ---------------------------------------- |
| Чат (личные сообщения, групповые)        | Мгновенная доставка, низкая задержка                 | [[URLSessionWebSocketTask]] / Starscream |
| Live-обновления (цены, котировки, спорт) | Реал-тайм без задержек                               | [[Starscream]] (удобнее)                 |
| Мультиплеерные игры (синхронизация)      | Быстрая передача координат, действий                 | Starscream                               |
| IoT / умные устройства                   | Постоянное соединение, push от устройства            | URLSessionWebSocketTask                  |
| Уведомления внутри приложения            | Альтернатива FCM/[[APNs]] для кастомных событий      | Starscream                               |
| Коллаборативные редакторы                | Синхронизация изменений в реальном времени           | Starscream                               |

### 2. Сравнение двух основных инструментов (2026)

| Характеристика               | URLSessionWebSocketTask (нативный, iOS 13+) | Starscream (сторонняя библиотека)    |
| ---------------------------- | ------------------------------------------- | ------------------------------------ |
| Зависимости                  | Нет (встроен в Foundation)                  | +1 [[SPM]]-пакет                     |
| Синтаксис                    | Низкоуровневый (completion + receive)       | Высокоуровневый (delegate + onEvent) |
| Авто-реконнект               | Нет (нужно писать вручную)                  | Да (встроенный + кастомный)          |
| Прогресс / большие сообщения | Ограничено                                  | Полная поддержка                     |
| Бинарные данные              | Да                                          | Да + удобные методы                  |
| Поддержка ping/pong          | Ручная                                      | Автоматическая                       |
| Закрытие с причиной          | Да (closeCode, reason)                      | Да + читаемые события                |
| Производительность           | Максимальная (нативный NIO)                 | Очень высокая                        |
| Рекомендация 2026            | Простые случаи, минимализм                  | Всё остальное (чат, игры, live)      |

**Вывод 2026**:  
- Простой echo / тестовый сервер → `URLSessionWebSocketTask`  
- Реальный чат, live-обновления, IoT → **Starscream** (или Vapor на сервере)

### 3. Полные примеры кода

#### Пример 1 — URLSessionWebSocketTask (нативный, минималистичный)

```swift
import Foundation

class NativeWebSocket: NSObject, URLSessionWebSocketDelegate {
    private var task: URLSessionWebSocketTask?
    private let url = URL(string: "wss://echo.websocket.org")!
    
    func connect() {
        let session = URLSession(configuration: .default, delegate: self, delegateQueue: nil)
        task = session.webSocketTask(with: url)
        task?.resume()
        receive()
    }
    
    func send(text: String) {
        let message = URLSessionWebSocketTask.Message.string(text)
        task?.send(message) { error in
            if let error {
                print("Send error:", error.localizedDescription)
            }
        }
    }
    
    private func receive() {
        task?.receive { [weak self] result in
            switch result {
            case .success(let message):
                switch message {
                case .string(let text):
                    print("Получено:", text)
                case .data(let data):
                    print("Бинарные данные:", data.count, "байт")
                @unknown default:
                    break
                }
                self?.receive() // рекурсия
                
            case .failure(let error):
                print("Receive error:", error.localizedDescription)
            }
        }
    }
    
    func disconnect() {
        task?.cancel(with: .goingAway, reason: nil)
    }
    
    // MARK: - Delegate
    
    func urlSession(_ session: URLSession,
                    webSocketTask: URLSessionWebSocketTask,
                    didOpenWithProtocol protocol: String?) {
        print("Соединение открыто, протокол:", protocol ?? "—")
    }
    
    func urlSession(_ session: URLSession,
                    webSocketTask: URLSessionWebSocketTask,
                    didCloseWith closeCode: URLSessionWebSocketTask.CloseCode,
                    reason: Data?) {
        let reasonStr = reason.flatMap { String(data: $0, encoding: .utf8) } ?? "—"
        print("Соединение закрыто, код:", closeCode.rawValue, "причина:", reasonStr)
    }
}

// Использование
let ws = NativeWebSocket()
ws.connect()
ws.send(text: "Привет, эхо!")
```

#### Пример 2 — Starscream (рекомендуемый для большинства случаев)

```swift
import Starscream

class ChatManager: WebSocketDelegate {
    private var socket: WebSocket!
    
    init() {
        var request = URLRequest(url: URL(string: "wss://echo.websocket.org")!)
        request.timeoutInterval = 5
        socket = WebSocket(request: request)
        socket.delegate = self
        socket.connect()
    }
    
    func send(text: String) {
        socket.write(string: text)
    }
    
    func send(data: Data) {
        socket.write(data: data)
    }
    
    // MARK: - WebSocketDelegate
    
    func didReceive(event: WebSocketEvent, client: WebSocket) {
        switch event {
        case .connected(let headers):
            print("Соединение установлено:", headers)
            
        case .disconnected(let reason, let code):
            print("Разрыв соединения:", reason, "код:", code)
            // Авто-реконнект
            DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
                client.connect()
            }
            
        case .text(let string):
            print("Текст:", string)
            
        case .binary(let data):
            print("Бинарные данные:", data.count, "байт")
            
        case .ping:
            print("Ping от сервера")
            
        case .pong:
            print("Pong от сервера")
            
        case .error(let error):
            print("Ошибка:", error?.localizedDescription ?? "—")
            
        case .cancelled:
            print("Соединение отменено")
            
        case .viabilityChanged(let isViable):
            print("Соединение жизнеспособно:", isViable)
            
        @unknown default:
            break
        }
    }
}

// Использование
let chat = ChatManager()
chat.send(text: "Привет, сервер!")
```

#### Пример 3 — Авто-реконнект + JWT-аутентификация (реальный чат)

```swift
class AuthenticatedWebSocket: WebSocketDelegate {
    private var socket: WebSocket!
    private let url: URL
    private let token: String
    private var reconnectAttempts = 0
    private let maxAttempts = 5
    
    init(url: URL, token: String) {
        self.url = url
        self.token = token
        super.init()
        connect()
    }
    
    private func connect() {
        var request = URLRequest(url: url)
        request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        
        socket = WebSocket(request: request)
        socket.delegate = self
        socket.connect()
    }
    
    // MARK: - Delegate
    
    func didReceive(event: WebSocketEvent, client: WebSocket) {
        switch event {
        case .connected:
            reconnectAttempts = 0
            print("WebSocket аутентифицирован")
            
        case .disconnected:
            if reconnectAttempts < maxAttempts {
                reconnectAttempts += 1
                print("Переподключение... попытка \(reconnectAttempts)")
                DispatchQueue.main.asyncAfter(deadline: .now() + Double(reconnectAttempts)) {
                    self.connect()
                }
            }
            
        case .text(let string):
            print("Сообщение:", string)
            
        case .error(let error):
            print("Ошибка:", error?.localizedDescription ?? "—")
            
        default:
            break
        }
    }
}
```

### 5. Таблица: сравнение URLSessionWebSocketTask vs Starscream

| Характеристика                       | URLSessionWebSocketTask                  | Starscream                                   |
|--------------------------------------|------------------------------------------|----------------------------------------------|
| Зависимости                          | Нет                                      | +1 SPM-пакет                                 |
| Синтаксис                            | Низкоуровневый (receive + send)          | Высокоуровневый (delegate + onEvent)         |
| Авто-реконнект                       | Нужно писать вручную                     | Встроенный + кастомный                       |
| Прогресс / большие сообщения         | Ограничено                               | Полная поддержка                             |
| Ping/Pong                            | Ручная                                   | Автоматическая                               |
| Закрытие с причиной                  | Да                                       | Да + читаемые события                        |
| Поддержка iOS                        | iOS 13+                                  | iOS 9+                                       |
| Рекомендация 2026                    | Простые случаи                           | Чаты, live, игры, IoT                        |

### 6. Лучшие практики 2026 года

- Используй **одну задачу** на соединение — не пересоздавай при каждом сообщении
- Всегда вызывай `receive()` рекурсивно или в цикле
- Для reconnect — экспоненциальная задержка (3с → 6с → 12с → 24с → ...)
- Для аутентификации — JWT в query или в первом сообщении
- Для бинарных данных — `Message.data`
- Для больших сообщений — разбивай на части
- Для UI — диспатчь на main
- Для отладки — реализуй `didOpenWithProtocol` и `didCloseWith`
- Для продакшена — добавь heartbeat (ping каждые 30–60 сек)

**Короткий девиз**:
> «WebSocket в 2026 — это реал-тайм без polling.  
> Простой случай → URLSessionWebSocketTask.  
> Чат, live, игры → Starscream + авто-реконнект + JWT.»
