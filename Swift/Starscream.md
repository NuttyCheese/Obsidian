## 📘 Определение

Starscream — сторонняя Swift‑библиотека, реализующая [[WebSocket]] (RFC 6455), то есть обеспечивает двустороннюю связь между клиентом (например, [[iOS]]‑приложением) и сервером через WebSocket‑соединение. Это даёт возможность получать и отправлять данные в реальном времени, без необходимости многократных [[HTTP]]‑запросов. ([GitHub](https://github.com/daltoniam/Starscream?utm_source=chatgpt.com "GitHub - daltoniam/Starscream: Websockets in swift for iOS and OSX"))

### Особенности

- Поддержка TLS/WSS (то есть защищённые WebSocket-соединения). ([GitHub](https://github.com/daltoniam/Starscream?utm_source=chatgpt.com "GitHub - daltoniam/Starscream: Websockets in swift for iOS and OSX"))
    
- Поддержка расширений (compression, ping/pong, прочее) в соответствии со стандартами. ([GitHub](https://github.com/daltoniam/Starscream?utm_source=chatgpt.com "GitHub - daltoniam/Starscream: Websockets in swift for iOS and OSX"))
    
- Работает асинхронно, на потоках фона через [[GCD]], не блокируя UI. ([GitHub](https://github.com/daltoniam/Starscream?utm_source=chatgpt.com "GitHub - daltoniam/Starscream: Websockets in swift for iOS and OSX"))
    

---

## 🔹 Примеры кода

```swift
import Starscream

class WebSocketManager: WebSocketDelegate {
    var socket: WebSocket?

    func connect() {
        var request = URLRequest(url: URL(string: "wss://example.com/socket")!)
        request.timeoutInterval = 5
        socket = WebSocket(request: request)
        socket?.delegate = self
        socket?.connect()
    }

    func disconnect() {
        socket?.disconnect()
    }

    // MARK: - WebSocketDelegate

    func didReceive(event: WebSocketEvent, client: WebSocket) {
        switch event {
        case .connected(let headers):
            print("✅ WebSocket connected: \(headers)")
        case .disconnected(let reason, let code):
            print("❌ WebSocket disconnected: \(reason) with code \(code)")
        case .text(let string):
            print("🟢 Received text: \(string)")
        case .binary(let data):
            print("🟡 Received data: \(data.count) bytes")
        case .error(let error):
            print("⚠️ Error: \(error?.localizedDescription ?? "")")
        default:
            break
        }
    }

    func send(message: String) {
        socket?.write(string: message)
    }
}
```

```swift
// Использование

let manager = WebSocketManager()
manager.connect()

// Позднее — отправить сообщение:
manager.send(message: "{\"action\":\"ping\"}")
```
