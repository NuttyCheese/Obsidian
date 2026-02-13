Вот **полное, подробное и максимально актуальное** (на 2026 год) руководство по **Stub** (заглушкам) в unit-тестировании на Swift — с лучшими практиками, примерами и сравнением подходов.

### 1. Что такое Stub и чем он отличается от других тестовых двойников

**Stub** — это **тестовый двойник** (test double), который:

- **возвращает заранее заданные значения** (или ошибки)  
- **не проверяет** вызовы методов (не считает их, не проверяет аргументы)  
- **используется только для изоляции** тестируемого класса от внешних зависимостей

**Ключевые отличия от других двойников** (2026):

| Вид двойника | Возвращает фиксированные данные? | Записывает/проверяет вызовы? | Проверяет аргументы? | Самый частый сценарий 2026 |
|--------------|-----------------------------------|-------------------------------|-----------------------|-----------------------------|
| **Dummy**    | Нет (пустышка)                    | Нет                           | Нет                   | Когда объект нужен, но не используется |
| **Stub**     | **Да**                            | **Нет**                       | **Нет**               | Подмена ответов сети/БД/сервисов |
| **Mock**     | Да (можно настроить)              | **Да**                        | **Да**                | Проверка вызовов и аргументов |
| **Spy**      | Да (реальная логика + запись)     | **Да**                        | **Да**                | Проверка + сохранение поведения |
| **Fake**     | Да (упрощённая реализация)        | Нет                           | Нет                   | In-memory БД, фейковая сеть |

**Золотое правило 2026**:
> Используй **Stub**, когда тебе нужно **только подменить ответ** (данные, успех/ошибка).  
> Используй **Mock**, когда важно **проверить, что метод был вызван** (и с какими аргументами).

### 2. Самые популярные способы создания Stub в Swift 2026

#### Способ 1 — Ручной Stub (самый надёжный и читаемый)

```swift
protocol NetworkService {
    func fetchUsers() async throws -> [User]
}

class NetworkServiceStub: NetworkService {
    // Фиксированные данные для теста
    var stubbedUsers: [User] = [
        User(id: 1, name: "Alice"),
        User(id: 2, name: "Bob")
    ]
    
    var shouldThrowError = false
    var errorToThrow: Error = NSError(domain: "TestError", code: -1)
    
    func fetchUsers() async throws -> [User] {
        if shouldThrowError {
            throw errorToThrow
        }
        return stubbedUsers
    }
}

// Тест
func testUserListViewModelLoadsUsersSuccessfully() async throws {
    let stub = NetworkServiceStub()
    let viewModel = UserListViewModel(service: stub)
    
    await viewModel.loadUsers()
    
    XCTAssertEqual(viewModel.users.count, 2)
    XCTAssertEqual(viewModel.users.first?.name, "Alice")
}

func testUserListViewModelHandlesError() async throws {
    let stub = NetworkServiceStub()
    stub.shouldThrowError = true
    
    let viewModel = UserListViewModel(service: stub)
    
    await viewModel.loadUsers()
    
    XCTAssertTrue(viewModel.hasError)
    XCTAssertEqual(viewModel.errorMessage, "TestError")
}
```

#### Способ 2 — Stub с помощью библиотек (Cuckoo / Mockingbird)

```swift
// Cuckoo пример (автогенерация моков)
let mockNetwork = mock(NetworkService.self)

stub(mockNetwork) { stub in
    when(stub.fetchUsers()).thenReturn([User(id: 1, name: "Test")])
    // или thenThrow(Error)
}

// Тест
let viewModel = UserListViewModel(service: mockNetwork as! any NetworkService)

await viewModel.loadUsers()

XCTAssertEqual(viewModel.users.count, 1)
XCTAssertEqual(viewModel.users.first?.name, "Test")
```

**Когда использовать библиотеки**:
- проект большой  
- много зависимостей  
- нужно много моков/стабов  
- хочешь `verify` + `stub` в одном инструменте

### 3. Лучшие практики использования Stub в Swift 2026

- **Stub только для изоляции** — не проверяй в нём вызовы (для этого Mock)  
- **Делай stub настраиваемым** — свойства `stubbedResult`, `shouldThrow`, `delay` и т.д.  
- **Async/await** — всегда используй `async throws` в протоколах и стабоах  
- **@MainActor** — если тестируешь ViewModel → тест тоже `@MainActor`  
- **Swift Testing** (Xcode 16+) — используй `#expect(await vm.users.count == 5)`  
- **Не мокай/стаби простые типы** (String, Int, Date, URL) — используй реальные  
- **YAGNI в тестах** — не пиши stub на то, что не используется в тесте  
- **Именование** — `NetworkServiceHappyPathStub`, `NetworkServiceErrorStub`, `EmptyUsersStub`  
- **Swift 6 strict concurrency** — stub должен быть Sendable или actor-safe

### 4. Когда использовать Stub (а когда Mock / Fake)

| Сценарий                                      | Лучший двойник | Почему |
|-----------------------------------------------|----------------|--------|
| Нужно подменить ответ сети (успех/ошибка)     | Stub           | Просто и быстро |
| Нужно проверить, что `trackEvent` был вызван  | Mock           | Нужны assertions на вызовы |
| Нужно протестировать работу с in-memory БД    | Fake           | Нужна полноценная имитация |
| Нужно просто передать объект, который не используется | Dummy      | Минимум кода |
| Нужно проверить реальное поведение + записать вызовы | Spy     | Редко, сложнее |

### 5. Итог: Stub — это самый простой и частый тестовый двойник

- **Stub** — когда нужно **только подменить ответ**  
- **Mock** — когда нужно **проверить вызовы**  
- **Fake** — когда нужна **полноценная имитация**  
- **В 90% случаев** в iOS unit-тестах достаточно **ручного Stub** — быстро, читаемо, без внешних зависимостей

**Короткий девиз 2026**:
> «Stub — это когда ты говоришь тесту: «не ходи в сеть, не ходи в БД — вот тебе готовые данные».  
> Простой, быстрый и надёжный способ изолировать логику от внешнего мира.»

Удачи с чистыми, быстрыми и изолированными тестами в Swift! 🧪