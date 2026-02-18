**`fileprivate`** — это модификатор доступа в Swift, который ограничивает видимость элемента **только внутри текущего файла** (.swift).

### Коротко и чётко

| Модификатор     | Видимость                                      | Где можно использовать элемент                  |
|-----------------|------------------------------------------------|-------------------------------------------------|
| `private`       | Только внутри текущей структуры/класса/enum    | Внутри `{ }` одного типа                        |
| `fileprivate`   | Весь файл (.swift), в котором объявлен элемент | В любом месте этого же файла                    |
| `internal`      | Весь модуль (по умолчанию)                     | Весь target/framework                           |
| `public`        | Везде, включая другие модули                   | Доступен для импорта и использования снаружи   |
| `open`          | Как `public` + можно наследовать/переопределять| Только для классов и методов                    |

### Когда использовать `fileprivate` (реальные сценарии 2025–2026)

| Ситуация                                      | Почему именно `fileprivate`                              | Альтернатива (когда не использовать)             |
|-----------------------------------------------|----------------------------------------------------------|--------------------------------------------------|
| Вспомогательные типы/функции в одном файле    | Не хочется засорять область видимости модуля             | `private` — если используется только внутри типа |
| Константы / конфиги, которые используются в нескольких типах одного файла | Удобно делить без создания отдельного файла              | `internal` — если нужно в других файлах модуля   |
| Временные / отладочные свойства/методы        | Видны только разработчику, который работает с файлом     | `private` — если не нужны вне типа               |
| Реализация протоколов в одном файле           | Несколько типов реализуют один протокол → общие хелперы   | `private` — если хелперы локальны для типа       |
| Один файл = один фичер (feature folder)       | Всё, что относится к одной фиче, остаётся внутри файла   | `internal` — если фича разбита по файлам         |

### Примеры (современный стиль 2026)

#### Пример 1. Вспомогательные типы в одном файле

```swift
// File: ProfileViewModel.swift

// fileprivate — видно только в этом файле
fileprivate struct ProfileUpdateRequest {
    let userId: String
    let name: String?
    let avatar: Data?
}

final class ProfileViewModel {
    func updateProfile(name: String?, avatar: Data?) async throws {
        let request = ProfileUpdateRequest(
            userId: currentUser.id,
            name: name,
            avatar: avatar
        )
        // ...
    }
}
```

#### Пример 2. Общие константы и хелперы

```swift
// File: AuthCoordinator.swift

fileprivate let defaultTimeout: TimeInterval = 30
fileprivate let maxRetryCount = 3

fileprivate func makeAuthError(from statusCode: Int) -> Error {
    switch statusCode {
    case 401: return AuthError.unauthorized
    case 403: return AuthError.forbidden
    default:  return AuthError.server(statusCode: statusCode)
    }
}

final class AuthCoordinator {
    func login() async throws {
        // используем fileprivate константы и функции
    }
}
```

#### Пример 3. fileprivate vs private (классическая путаница)

```swift
class UserViewModel {
    private var internalCache: [String: Data] = [:]          // только внутри UserViewModel
    fileprivate var sharedLogger: Logger?                    // видно во всём файле
    
    fileprivate func logEvent(_ message: String) {
        sharedLogger?.info(message)
    }
}

class UserProfileViewModel {  // в том же файле
    func refresh() {
        // может вызвать logEvent()
    }
}
```

### 4. Лучшие практики fileprivate в Swift 2026

- Используй `fileprivate`, когда несколько типов/функций/констант **логически связаны** и находятся в **одном файле**  
- Держи файл **не больше 400–600 строк** — если файл разрастается → лучше вынести в отдельный файл и сделать `internal`/`private`  
- В **SwiftUI** — часто используй `fileprivate` для preview-констант, моделей внутри одного файла  
- В **feature-based структуре** (по фичам) — `fileprivate` идеально подходит для внутренних хелперов одной фичи  
- **Не используй** `fileprivate` для публичных API или для типов, которые могут понадобиться в других файлах модуля  
- **Swift 6 strict concurrency** — `fileprivate` никак не влияет на Sendable/акторы — это только область видимости  
- **Документируйте** — пиши комментарий «fileprivate — общие константы и хелперы только для этого файла»

**Короткий девиз 2026**:
> `fileprivate` — это «видно только в этом файле, и нигде больше».  
> Используй его, когда:  
> - несколько типов/функций работают вместе  
> - не хочешь засорять модуль публичными/внутренними именами  
> - файл = логическая единица (одна фича, один экран, один сервис)  
> Это **золотая середина** между `private` и `internal`.

Удачи с чистой и логичной видимостью в твоём проекте! 📄