#system_design
## Определение

**gRPC** — это современный фреймворк для удалённых вызовов процедур (Remote Procedure Call, RPC), который позволяет клиенту и серверу **обмениваться данными эффективно и строго типизировано**.

> gRPC использует протокол HTTP/2 и **Protocol Buffers (protobuf)** для сериализации данных, что делает его быстрым и лёгким для передачи.

---

## Основные компоненты gRPC

1. **Service (Сервис)**
    
    - Определяет набор методов, которые сервер предоставляет клиенту.
        
    - Пример:
        
    
    ```proto
    service UserService {
        rpc GetUser (UserRequest) returns (UserResponse);
    }
    ```
    
2. **Messages (Сообщения)**
    
    - Структуры данных, передаваемые между клиентом и сервером.
        
    - Пример:
        
    
    ```proto
    message UserRequest {
        string id = 1;
    }
    
    message UserResponse {
        string name = 1;
        string email = 2;
    }
    ```
    
3. **Методы RPC**
    
    - **Unary RPC** — один запрос, один ответ.
        
    - **Server streaming RPC** — сервер отправляет несколько ответов на один запрос.
        
    - **Client streaming RPC** — клиент отправляет несколько запросов, сервер один ответ.
        
    - **Bidirectional streaming RPC** — обмен данными в обе стороны одновременно.
        

---

## Преимущества gRPC

1. **Высокая производительность**
    
    - HTTP/2 → мультиплексирование, меньшая задержка.
        
    - Protocol Buffers → компактная бинарная сериализация.
        
2. **Сильная типизация**
    
    - Ошибки в структуре данных обнаруживаются на этапе компиляции.
        
3. **Поддержка стриминга и реального времени**
    
    - Сервер может отправлять данные по мере их появления.
        
4. **Межъязыковая совместимость**
    
    - Сервер и клиент могут быть на разных языках: Swift, Kotlin, Python, Go, C# и т.д.
        
5. **Контракты через .proto**
    
    - Клиент и сервер используют одну схему, что упрощает интеграцию и обновления API.
        

---

## gRPC vs REST

|Критерий|REST|gRPC|
|---|---|---|
|Протокол|HTTP/1.1|HTTP/2|
|Формат данных|JSON|Protocol Buffers (бинарный)|
|Типизация|Нет|Сильная|
|Поток данных|Нет|Поддержка стриминга (server/client/bidirectional)|
|Производительность|Средняя|Высокая (меньше задержка, меньше трафика)|

---

## Использование gRPC в iOS

### Подготовка

1. Создаём `.proto` файлы с описанием сервисов и сообщений.
    
2. Генерируем Swift код через **protoc plugin for Swift**.
    

### Пример клиента на Swift

```swift
import GRPC
import NIO

class UserServiceClientWrapper {
    private let client: UserService_UserServiceClient
    private let group = MultiThreadedEventLoopGroup(numberOfThreads: 1)
    
    init() {
        let channel = ClientConnection.insecure(group: group)
            .connect(host: "example.com", port: 50051)
        client = UserService_UserServiceClient(channel: channel)
    }
    
    func getUser(id: String) {
        let request = UserRequest.with {
            $0.id = id
        }
        
        let call = client.getUser(request)
        call.response.whenComplete { result in
            switch result {
            case .success(let response):
                print("User name: \(response.name)")
            case .failure(let error):
                print("Ошибка gRPC: \(error)")
            }
        }
    }
    
    func shutdown() throws {
        try group.syncShutdownGracefully()
    }
}
```

---

## Применение gRPC в мобильных приложениях

- **Чат-приложения** → bidirectional streaming для сообщений в реальном времени.
    
- **Приложения доставки** → server streaming для обновления статусов заказов.
    
- **Финансовые приложения** → низкая задержка при обмене котировками.
    
- **Игры онлайн** → быстрый обмен состоянием игроков.
    

---

## Преимущества для iOS

- Меньше трафика по сравнению с REST (JSON → бинарный protobuf).
    
- Поддержка стриминга в реальном времени через HTTP/2.
    
- Возможность легко кэшировать ответы на клиенте, если нужно.
    

---

## Итог

- **gRPC** — современный, быстрый и строго типизированный способ обмена данными между сервером и клиентом.
    
- Подходит для приложений с высокой нагрузкой и необходимостью реального времени.
    
- В iOS реализуется через **Swift gRPC**, поддерживает unary и streaming вызовы.
    
- Отличие от REST: **бинарные данные, HTTP/2, стриминг, высокая производительность**.
    

---
