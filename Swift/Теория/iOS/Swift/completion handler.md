**Completion handler** (или просто **completion**) — это **замыкание** (closure), которое передаётся в функцию как последний аргумент и вызывается **после завершения** какой-либо операции (обычно асинхронной), чтобы сообщить результат, успех/ошибку или просто факт завершения.

В 2026 году completion handler всё ещё очень часто встречается в UIKit, Foundation и legacy-коде, хотя большинство новых API уже перешли на **async/await**.

### 1. Почему completion handler до сих пор важен

| Сценарий                                      | Почему completion всё ещё используется                   | Современная альтернатива (2026)                     |
|-----------------------------------------------|----------------------------------------------------------|------------------------------------------------------|
| UIKit анимации                                | `UIView.animate` и `UIView.transition`                   | `withAnimation { ... }` + `Task`                     |
| URLSession (старый API)                       | `dataTask`, `uploadTask`, `downloadTask`                 | `URLSession.shared.data(from:)` — async версия       |
| Core Location, Core Motion, Core Bluetooth    | `requestLocation`, `startUpdatingLocation` и т.д.        | Некоторые методы уже async, но большинство — completion |
| FileManager, PHPhotoLibrary, AVFoundation     | `moveItem`, `requestAuthorization`, `exportAsynchronously` | Частично async, частично completion                  |
| Legacy-библиотеки и SDK (Firebase, Alamofire до 5.x) | Большинство SDK до сих пор на completion                | Новые версии — async/await                           |
| Кастомные асинхронные операции                 | Когда пишешь свою асинхронную функцию на GCD/Operation   | Переписывай на `async throws`                        |

### 2. Самый современный и рекомендуемый паттерн 2026 года

#### Вариант 1: Классический completion handler (ещё очень живой в UIKit)

```swift
func loadUserProfile(userId: String, completion: @escaping (Result<User, Error>) -> Void) {
    // Симуляция сетевого запроса
    DispatchQueue.global().asyncAfter(deadline: .now() + 1.5) {
        if Bool.random() {
            let user = User(id: userId, name: "Alice")
            completion(.success(user))
        } else {
            completion(.failure(NSError(domain: "Auth", code: -1009)))
        }
    }
}

// Использование в UIViewController
class ProfileViewController: UIViewController {
    
    func fetchProfile() {
        showLoadingIndicator()
        
        loadUserProfile(userId: "123") { [weak self] result in
            guard let self else { return }
            
            DispatchQueue.main.async {
                self.hideLoadingIndicator()
                
                switch result {
                case .success(let user):
                    self.updateUI(with: user)
                case .failure(let error):
                    self.showError(error.localizedDescription)
                }
            }
        }
    }
}
```

#### Вариант 2: Обёртка старого completion в async/await (золотой стандарт миграции)

```swift
func loadUserProfile(userId: String) async throws -> User {
    try await withCheckedThrowingContinuation { continuation in
        loadUserProfile(userId: userId) { result in
            switch result {
            case .success(let user):
                continuation.resume(returning: user)
            case .failure(let error):
                continuation.resume(throwing: error)
            }
        }
    }
}

// Использование — чистый и линейный код
Task {
    do {
        let user = try await loadUserProfile(userId: "123")
        await updateUI(with: user)
    } catch {
        await showError(error.localizedDescription)
    }
}
```

**Преимущества обёртки**:
- Код становится линейным (без вложенности)
- Можно использовать `try await`, `async let`, `TaskGroup`
- Легко тестировать с `XCTest` + `async`
- Совместимо с Swift 6 strict concurrency

### 3. Лучшие практики completion handler в Swift 2026

- **Всегда используй `[weak self]`** в escaping completion (иначе retain cycle)
- **Передавай результат через `Result<Success, Error>`** — это стандарт де-факто
- **Диспатчи на главный поток** для обновления UI: `DispatchQueue.main.async` или `await MainActor.run`
- **Не забывай отменять операции** при `deinit` или уходе с экрана (особенно таймеры, запросы, location)
- **Оборачивай legacy-API в async** с помощью `withCheckedThrowingContinuation` — это must-have для новых проектов
- **Swift 6 strict concurrency** — completion handler должен захватывать `self` явно (`[weak self]`), иначе предупреждение/ошибка
- **Документируйте** — пиши комментарий «@escaping completion — вызывается после загрузки профиля на фоне»

**Короткий девиз 2026**:
> Completion handler — это **closure**, который говорит функции: «когда закончишь — позвони мне и скажи результат».  
> В 2026 году:  
> - используй `Result` + `@escaping`  
> - всегда `[weak self]` + `guard let self`  
> - оборачивай старые API в `async throws`  
> - новый код пиши на `async/await` — callback hell мёртв.

Удачи с чистым и современным асинхронным кодом без вложенных замыканий! 🚀