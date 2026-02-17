#system_design
## Определение

**Unit-тестирование** — это метод тестирования, при котором проверяется **отдельная единица кода** (функция, метод, класс) на корректность работы.

Цель:

- Убедиться, что **логика работает правильно**.
    
- Быстро находить **ошибки в коде**.
    
- Обеспечить **безопасность рефакторинга**.
    

Для iOS это обычно тесты:

- **ViewController / Presenter / Interactor** (в [[Clean Swift (VIP) Architecture]] или [[MVVM (Model-View-ViewModel) Architecture]]).
    
- **Сервисы** (NetworkService, DataService).
    
- **Утилитарные функции** (валидация, вычисления).
    

---

## 1. Принцип работы XCTest

1. Создаётся тестовый класс, наследующийся от `XCTestCase`.
    
2. В нём определяются тестовые методы с префиксом `test`.
    
3. В каждом методе:
    
    - Подготавливаются входные данные.
        
    - Вызывается тестируемая функция.
        
    - Сравниваются результаты с ожидаемыми через **assertions**.
        

```swift
import XCTest
@testable import MyApp

class MathUtilsTests: XCTestCase {

    func testSum() {
        let result = MathUtils.sum(2, 3)
        XCTAssertEqual(result, 5, "Сумма 2 + 3 должна быть 5")
    }
}
```

---

## 2. Основные assertions

|Assertion|Описание|
|---|---|
|`XCTAssertTrue(_:)`|Проверка, что условие истинно|
|`XCTAssertFalse(_:)`|Проверка, что условие ложно|
|`XCTAssertEqual(_:_:)`|Проверка равенства значений|
|`XCTAssertNotNil(_:)`|Проверка, что объект не nil|
|`XCTAssertNil(_:)`|Проверка, что объект nil|
|`XCTAssertThrowsError(_:)`|Проверка, что функция выбрасывает ошибку|
|`XCTAssertNoThrow(_:)`|Проверка, что функция не выбрасывает ошибку|

---

## 3. Mocking и Dependency Injection

Для unit-тестов важно **изолировать код**:

- Зависимости (NetworkService, Database) заменяются на **mock-объекты**.
    
- Используется **Dependency Injection** для передачи моков.
    

```swift
class MockNetworkService: NetworkServiceProtocol {
    func fetchData(completion: (Result<String, Error>) -> Void) {
        completion(.success("Mocked data"))
    }
}

func testInteractorFetch() {
    let mockService = MockNetworkService()
    let interactor = MyInteractor(service: mockService)

    interactor.fetchData()
    XCTAssertEqual(interactor.data, "Mocked data")
}
```

---

## 4. Asynchronous тесты

- Для асинхронных операций ([[URLSession]], [[Core Data]], [[Combine]]`) используется **expectation** и` waitForExpectations(timeout:)`.
    

```swift
func testAsyncFetch() {
    let expectation = self.expectation(description: "Fetch data")
    let service = NetworkService()

    service.fetchData { result in
        XCTAssertEqual(result, "Expected data")
        expectation.fulfill()
    }

    waitForExpectations(timeout: 5, handler: nil)
}
```

---

## 5. Best Practices

1. **Каждый тест — изолирован и независим**.
    
2. **Один тест = одна проверка** (Single Responsibility Principle).
    
3. **Использовать mock и stub объекты** для зависимостей.
    
4. **Покрывать крайние случаи** (empty data, [[nil]], ошибки).
    
5. **Интеграция с [[CI]]/[[CD]]** для автоматического запуска тестов при пуше.
    
6. **Не тестировать UI напрямую** — для этого есть UI и snapshot тесты.
    

---

## 6. Итог

- **XCTest** — стандартный фреймворк для unit-тестов в iOS.
    
- Позволяет тестировать **логику компонентов и сервисов** без запуска UI.
    
- **Асинхронные тесты** и **mocking** помогают изолировать зависимости и проверить корректность работы кода.
    
- В сочетании с **UI tests** и **snapshot testing** обеспечивает комплексное покрытие приложения тестами.
    

---
