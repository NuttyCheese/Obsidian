**`URLSessionWebSocketTask`** — это специализированная задача (`URLSessionTask`) внутри [[URLSession]], предназначенная для **установки, поддержания и управления WebSocket-соединениями**.

Это **нативный** способ работы с [[WebSocket]] в [[iOS]]/macOS без сторонних библиотек (в отличие от Socket.IO или [[Starscream]]).

### 1. Когда использовать URLSessionWebSocketTask

| Сценарий                                 | Рекомендуется ли | Почему                                                   |
| ---------------------------------------- | ---------------- | -------------------------------------------------------- |
| Чат в реальном времени                   | Да               | Двусторонняя связь, низкая задержка                      |
| Live-обновления (цены, котировки, спорт) | Да               | Подписка на события                                      |
| Мультиплеерные игры (синхронизация)      | Да               | Быстрая передача сообщений                               |
| IoT / умные устройства                   | Да               | Постоянное соединение                                    |
| Простой ping-pong / echo-сервер          | Да               | Простая реализация                                       |
| Сложные протоколы (Socket.IO, STOMP)     | Нет              | Лучше Starscream / Socket.IO-[[Swift]]                   |
| Высоконагруженный чат с комнатами        | Иногда           | Зависит от масштаба (лучше [[Vapor]] + WebSocket сервер) |

### 2. Основные методы и состояния

| Метод / Свойство              | Что делает                                                              | Когда вызывать / читать     |
| ----------------------------- | ----------------------------------------------------------------------- | --------------------------- |
| `resume()`                    | Открывает соединение                                                    | После создания задачи       |
| `send(_:completionHandler:)`  | Отправляет сообщение ([[String]] или data)                              | После открытия              |
| `receive(completionHandler:)` | Асинхронно получает следующее сообщение                                 | После открытия (рекурсивно) |
| `cancel(with:reason:)`        | Закрывает соединение с кодом и причиной                                 | При выходе из экрана        |
| `state`                       | Текущее состояние задачи (.running, .suspended, .canceling, .completed) | Отладка                     |
| `closeCode`                   | Код закрытия (после завершения)                                         | После закрытия              |
| `closeReason`                 | Причина закрытия ([[Data]]?)                                            | После закрытия              |

### 3. Полные примеры кода

#### Пример 1 — Простейший echo-клиент (классика)

```swift
import Foundation

class EchoClient: NSObject, URLSessionWebSocketDelegate {
    private var webSocketTask: URLSessionWebSocketTask?
    private let url = URL(string: "wss://echo.websocket.org")!
    
    func connect() {
        let session = URLSession(configuration: .default, delegate: self, delegateQueue: nil)
        webSocketTask = session.webSocketTask(with: url)
        webSocketTask?.resume()
        receiveMessage() // сразу начинаем слушать
    }
    
    func send(text: String) {
        let message = URLSessionWebSocketTask.Message.string(text)
        webSocketTask?.send(message) { error in
            if let error = error {
                print("Ошибка отправки:", error.localizedDescription)
            } else {
                print("Отправлено:", text)
            }
        }
    }
    
    private func receiveMessage() {
        webSocketTask?.receive { [weak self] result in
            switch result {
            case .success(let message):
                switch message {
                case .string(let text):
                    print("Получено:", text)
                case .data(let data):
                    print("Получены бинарные данные:", data.count, "байт")
                @unknown default:
                    break
                }
                self?.receiveMessage() // рекурсия — ждём следующее
                
            case .failure(let error):
                print("Ошибка соединения:", error.localizedDescription)
            }
        }
    }
    
    func disconnect() {
        webSocketTask?.cancel(with: .goingAway, reason: nil)
    }
    
    // MARK: - URLSessionWebSocketDelegate
    
    func urlSession(_ session: URLSession,
                    webSocketTask: URLSessionWebSocketTask,
                    didOpenWithProtocol protocol: String?) {
        print("WebSocket открыт, протокол:", protocol ?? "—")
    }
    
    func urlSession(_ session: URLSession,
                    webSocketTask: URLSessionWebSocketTask,
                    didCloseWith closeCode: URLSessionWebSocketTask.CloseCode,
                    reason: Data?) {
        let reasonStr = reason.flatMap { String(data: $0, encoding: .utf8) } ?? "—"
        print("WebSocket закрыт, код:", closeCode.rawValue, "причина:", reasonStr)
    }
}

// Использование
let client = EchoClient()
client.connect()
client.send(text: "Привет, эхо-сервер!")
// Через несколько секунд получим эхо-ответ
```

#### Пример 2 — Отправка и получение бинарных данных (например, фото в чате)

```swift
func sendImage(_ imageData: Data) {
    let message = URLSessionWebSocketTask.Message.data(imageData)
    webSocketTask?.send(message) { error in
        if let error = error {
            print("Ошибка отправки изображения:", error)
        } else {
            print("Изображение отправлено")
        }
    }
}

private func receiveMessage() {
    webSocketTask?.receive { [weak self] result in
        switch result {
        case .success(let message):
            switch message {
            case .data(let data):
                // Получено изображение
                if let image = UIImage(data: data) {
                    print("Получено изображение размером:", data.count, "байт")
                }
            case .string(let text):
                print("Текстовое сообщение:", text)
            @unknown default:
                break
            }
            self?.receiveMessage()
            
        case .failure(let error):
            print("Ошибка:", error)
        }
    }
}
```

#### Пример 3 — Автоматический reconnect при разрыве

```swift
class ReconnectingWebSocket: NSObject, URLSessionWebSocketDelegate {
    private var task: URLSessionWebSocketTask?
    private var url: URL
    private var reconnectAttempts = 0
    private let maxReconnectAttempts = 5
    
    init(url: URL) {
        self.url = url
        super.init()
    }
    
    func connect() {
        let session = URLSession(configuration: .default, delegate: self, delegateQueue: nil)
        task = session.webSocketTask(with: url)
        task?.resume()
        receiveMessage()
    }
    
    private func receiveMessage() {
        task?.receive { [weak self] result in
            guard let self else { return }
            
            switch result {
            case .success:
                self.reconnectAttempts = 0 // сброс счётчика
                self.receiveMessage() // продолжаем слушать
                
            case .failure(let error):
                print("Ошибка:", error.localizedDescription)
                self.reconnect()
            }
        }
    }
    
    private func reconnect() {
        guard reconnectAttempts < maxReconnectAttempts else {
            print("Достигнут лимит переподключений")
            return
        }
        
        reconnectAttempts += 1
        print("Переподключение... попытка \(reconnectAttempts)/\(maxReconnectAttempts)")
        
        DispatchQueue.global().asyncAfter(deadline: .now() + 3) { [weak self] in
            self?.connect()
        }
    }
    
    func urlSession(_ session: URLSession,
                    webSocketTask: URLSessionWebSocketTask,
                    didOpenWithProtocol protocol: String?) {
        print("Соединение открыто")
        reconnectAttempts = 0
    }
    
    func urlSession(_ session: URLSession,
                    webSocketTask: URLSessionWebSocketTask,
                    didCloseWith closeCode: URLSessionWebSocketTask.CloseCode,
                    reason: Data?) {
        print("Соединение закрыто, код:", closeCode.rawValue)
        reconnect()
    }
}
```

### 4. Таблица: коды закрытия (CloseCode)

| Код (rawValue) | Название                     | Когда используется                     | Что делать клиенту |
|----------------|------------------------------|----------------------------------------|---------------------|
| 1000           | `.normalClosure`             | Нормальное закрытие                    | Ничего, всё OK      |
| 1001           | `.goingAway`                 | Сервер уходит / страница закрыта       | Попытаться переподключиться |
| 1006           | (нет кода)                   | Соединение разорвано без причины       | Переподключиться    |
| 1008           | `.policyViolation`           | Нарушение политики                     | Проверить запрос    |
| 1011           | `.internalServerError`       | Ошибка сервера                         | Переподключиться позже |
| 1012           | `.serviceRestart`            | Сервер перезапускается                 | Переподключиться    |

### 5. Лучшие практики 2026 года

- Используй **одну задачу на соединение** — не создавай новую при каждом сообщении
- Всегда вызывай `receive()` рекурсивно или в цикле — иначе сообщения перестанут приходить
- Для reconnect — экспоненциальную задержку (3с → 6с → 12с → ...)
- Для аутентификации — отправляй токен в query или в первом сообщении
- Для бинарных данных — используй `Message.data`
- Для больших сообщений — разбивай на части
- Для фоновой работы — используй background-сессию
- Для UI-обновлений — всегда диспатчить на main
- Для отладки — реализуй `didOpenWithProtocol` и `didCloseWith`

**Короткий девиз**:
> «URLSessionWebSocketTask — это нативный, мощный и современный WebSocket-клиент в iOS.  
> В 2026 году — используй его для чатов, live-обновлений и IoT.  
> Главное: resume → receive рекурсивно → обрабатывай close и reconnect.»
