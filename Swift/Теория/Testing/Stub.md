**Stub (Заглушка):**

- Заглушка представляет собой простой объект, который предоставляет заранее определенные ответы на запросы.
- Его цель - предоставить реализацию методов или свойств для использования в тестах, чтобы изолировать компоненты кода от внешних зависимостей.
- Обычно используется для замещения сложных внешних зависимостей, таких как базы данных, сетевые запросы или внешние [[API]].
- Пример: создание заглушки для внешнего сервиса, который возвращает предопределенные данные.

Пример использования заглушки (Stub):

Предположим, у нас есть класс `NetworkService`, который выполняет сетевые запросы, и мы хотим протестировать класс `UserRepository`, который зависит от `NetworkService`. Мы создадим заглушку `NetworkServiceStub`, чтобы изолировать `UserRepository` от реальных сетевых запросов во время тестирования.
```swift
protocol NetworkService {
    func fetchData(completion: @escaping (Result<Data, Error>) -> Void)
}

class UserRepository {
    private let networkService: NetworkService
    
    init(networkService: NetworkService) {
        self.networkService = networkService
    }
    
    func getUserData(completion: @escaping (Result<Data, Error>) -> Void) {
        networkService.fetchData(completion: completion)
    }
}

// Заглушка NetworkService для тестирования UserRepository
class NetworkServiceStub: NetworkService {
    func fetchData(completion: @escaping (Result<Data, Error>) -> Void) {
        // Возвращаем заранее определенные данные для тестирования
        let testData = "Test data".data(using: .utf8)!
        completion(.success(testData))
    }
}

// Тестирование UserRepository с использованием заглушки
func testUserRepository() {
    let stub = NetworkServiceStub()
    let userRepository = UserRepository(networkService: stub)
    
    userRepository.getUserData { result in
        switch result {
        case .success(let data):
            print("Received user data: \(data)")
        case .failure(let error):
            print("Error fetching user data: \(error)")
        }
    }
}

```