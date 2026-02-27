## 1. Что такое gRPC

**gRPC** — это современный **фреймворк для удалённого вызова процедур (RPC)**, разработанный Google.

- RPC = Remote Procedure Call → позволяет вызвать метод на **другом сервере или устройстве**, как если бы это был локальный метод.
    
- gRPC работает поверх **HTTP/2**, использует **Protocol Buffers (protobuf)** для сериализации данных.
    

Проще говоря:

> gRPC позволяет клиенту вызвать метод на сервере, передав параметры и получив ответ, почти так же, как обычный метод в коде.

---

## 2. Почему gRPC

gRPC решает проблемы классических [[REST API]]:

| Проблема REST                            | Как gRPC решает                                        |
| ---------------------------------------- | ------------------------------------------------------ |
| [[JSON]] → текст, медленно и много места | Protobuf → бинарная сериализация, меньше байт, быстрее |
| [[HTTP]]/1.1 → один запрос за раз        | HTTP/2 → multiplexing, потоковые запросы, push         |
| Нет строгой схемы                        | Protobuf → строгая схема с типами                      |
| Много ручного кода                       | gRPC генерирует код клиента/сервера автоматически      |

---

## 3. Архитектура gRPC

gRPC строится вокруг нескольких ключевых компонентов:

1. **Service definition (proto файл)**
    
    - Определяет сервис, методы и типы данных
        
    - Например:
        

```proto
syntax = "proto3";

package example;

service UserService {
    rpc GetUser(UserRequest) returns (UserResponse);
    rpc CreateUser(CreateUserRequest) returns (UserResponse);
}

message UserRequest {
    int32 id = 1;
}

message CreateUserRequest {
    string name = 1;
    int32 age = 2;
}

message UserResponse {
    int32 id = 1;
    string name = 2;
    int32 age = 3;
}
```

2. **Code generation**
    
    - gRPC использует `protoc` для генерации кода **клиента и сервера** на [[Swift]], Java, Go, C# и др.
        
3. **Client**
    
    - Вызывает методы сервиса как локальные методы, но на самом деле данные сериализуются в protobuf и отправляются на сервер.
        
4. **Server**
    
    - Реализует методы сервиса, принимает запросы, десериализует protobuf и отправляет ответы.
        

---

## 4. Протокол передачи

gRPC использует **HTTP/2**, что даёт:

- **Multiplexing** → несколько запросов/ответов по одному TCP соединению
    
- **Server push** → сервер может «толкнуть» данные клиенту
    
- **Бинарная передача** → Protobuf → меньше трафика и быстрее парсинг
    

---

## 5. Типы RPC в gRPC

gRPC поддерживает четыре основных типа вызовов:

|Тип|Описание|Пример|
|---|---|---|
|**Unary**|Один запрос → один ответ|`GetUser`|
|**Server streaming**|Один запрос → поток ответов|`ListUsers`|
|**Client streaming**|Поток запросов → один ответ|`UploadLogs`|
|**Bidirectional streaming**|Поток запросов ↔ поток ответов|Чат, видео, IoT|

Пример **серверного стриминга** в proto:

```proto
rpc ListUsers(UserRequest) returns (stream UserResponse);
```

- Клиент делает один запрос, а сервер присылает несколько сообщений по одному.
    

---

## 6. Как это выглядит в [[Swift]]

### 6.1 Установка gRPC для Swift

- Используем **SwiftGRPC** или официальный `grpc-swift` через [[SPM]]:
    

```swift
dependencies: [
    .package(url: "https://github.com/grpc/grpc-swift.git", from: "1.15.0")
]
```

### 6.2 Генерация кода

```bash
protoc --swift_out=. --swiftgrpc_out=. user.proto
```

- Получаем Swift файлы: `UserServiceClient.swift`, `UserServiceServer.swift`
    

### 6.3 Пример Unary вызова

```swift
let channel = ClientConnection.insecure(group: MultiThreadedEventLoopGroup(numberOfThreads: 1))
    .connect(host: "localhost", port: 50051)

let client = UserServiceClient(channel: channel)

let request = UserRequest.with {
    $0.id = 123
}

let response = try client.getUser(request).response.wait()
print("User name:", response.name)
```

### 6.4 Пример серверного метода

```swift
class UserServiceImpl: UserServiceProvider {
    func getUser(request: UserRequest, context: StatusOnlyCallContext) -> EventLoopFuture<UserResponse> {
        let response = UserResponse.with {
            $0.id = request.id
            $0.name = "Alice"
            $0.age = 25
        }
        return context.eventLoop.makeSucceededFuture(response)
    }
}
```

- Всё синхронно выглядит как обычный метод, но внутри — сериализация, HTTP/2 и сетевой вызов.
    

---

## 7. Преимущества gRPC

1. **Высокая производительность** (HTTP/2 + protobuf)
    
2. **Типобезопасность** (Protobuf схемы)
    
3. **Кроссплатформенность** (Swift, Go, Java, C#, Python и др.)
    
4. **Поддержка стриминга** (streaming [[API]])
    
5. **Автогенерация кода** → меньше ручного boilerplate
    

---

## 8. Ограничения gRPC

- Клиенты должны поддерживать HTTP/2 → старые браузеры без gRPC-web не работают напрямую
    
- Кривая обучения выше, чем у REST
    
- Сложнее интегрировать с JSON/REST сервисами, нужен конвертер (gRPC-JSON transcoding)
    

---

## 9. gRPC vs [[REST]]

| Параметр           | REST                | gRPC                                |
| ------------------ | ------------------- | ----------------------------------- |
| Формат данных      | [[JSON]] (текст)    | Protobuf (бинарный)                 |
| HTTP версия        | 1.1                 | 2                                   |
| Тип вызова         | Только запрос-ответ | Unary + Streaming                   |
| Производительность | Ниже                | Выше (меньше байт, меньше парсинга) |
| Типизация          | Слабая              | Строгая (protobuf)                  |
| Code generation    | Обычно вручную      | Автоматически                       |

---

## 10. Использование в iOS / Swift

- **Микросервисы** → клиент [[iOS]] + сервер Go/Swift/Java
    
- **Realtime стриминг** → чаты, видео, IoT
    
- **Мобильные клиенты с низкой пропускной способностью** → protobuf меньше JSON по объёму
    
- **Типобезопасные API** → меньше runtime ошибок
    

---
