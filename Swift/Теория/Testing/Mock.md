**Mock** — это **тестовый двойник** (test double), который:

- имитирует поведение реального объекта  
- **записывает** все вызовы методов (кто вызвал, с какими аргументами, сколько раз)  
- позволяет **проверять взаимодействия** (assertions на вызовы)  
- возвращает **заранее подготовленные** данные или ошибки

**Главные цели мока в 2026**:

- изолировать тестируемый класс от внешних зависимостей (сеть, БД, аналитика, уведомления)  
- проверять **контракт взаимодействия** (метод вызван? с правильными параметрами? в правильном порядке?)  
- делать тесты **быстрыми** и **детерминированными** (не зависят от реального сервера или времени)  
- упрощать **тестирование edge-кейсов** (ошибки сети, таймауты, пустые списки)

### 2. Виды тестовых двойников (test doubles) — что выбрать в 2026

| Вид двойника | Что делает                                       | Проверяет поведение? | Проверяет состояние? | Самый частый инструмент 2026                 | Когда использовать                          |
| ------------ | ------------------------------------------------ | -------------------- | -------------------- | -------------------------------------------- | ------------------------------------------- |
| **Dummy**    | Просто заглушка (ничего не делает)               | Нет                  | Нет                  | Пустой класс / протокол                      | Когда объект нужен, но не используется      |
| **[[Stub]]** | Возвращает заранее заданные значения             | Нет                  | Да                   | `stub.returnValue = ...`                     | Когда нужно подменить ответ                 |
| **Mock**     | Записывает вызовы + проверяет их                 | Да                   | Да                   | [[XCTestExpectation]] / Cuckoo / Mockingbird | Когда важно **взаимодействие**              |
| **Spy**      | Записывает вызовы, но использует реальную логику | Да                   | Да                   | Ручная реализация + вызов super              | Когда нужно проверить + сохранить поведение |
| **Fake**     | Полноценная, но упрощённая реализация            | Нет                  | Да                   | [[In-memory]] БД, FakeNetworkClient          | Когда нужен реалистичный объект             |

**Золотое правило 2026**:
> Используй **Mock**, если хочешь проверить **вызовы методов** и **аргументы**.  
> Используй **Stub**, если нужно только **подменить ответ**.  
> Используй **Fake**, если нужна полноценная имитация (например, in-memory [[Core Data]]).

### 3. Самые популярные способы создания мока в Swift 2026

#### Способ 1 — Ручной Mock (самый надёжный и читаемый)

```swift
protocol AnalyticsTracking {
    func trackEvent(name: String, parameters: [String: Any]?)
    func trackScreen(_ name: String)
}

class AnalyticsTrackingMock: AnalyticsTracking {
    // Запись вызовов
    var trackEventCalls: [(name: String, parameters: [String: Any]?)] = []
    var trackScreenCalls: [String] = []
    
    // Счётчики (альтернатива массиву вызовов)
    var trackEventCallCount = 0
    var lastTrackedEvent: String?
    
    func trackEvent(name: String, parameters: [String: Any]?) {
        trackEventCalls.append((name, parameters))
        trackEventCallCount += 1
        lastTrackedEvent = name
    }
    
    func trackScreen(_ name: String) {
        trackScreenCalls.append(name)
    }
}

// Тест
func testButtonTapTracksEvent() {
    let mock = AnalyticsTrackingMock()
    let viewModel = ButtonViewModel(analytics: mock)
    
    viewModel.onButtonTap()
    
    XCTAssertEqual(mock.trackEventCallCount, 1)
    XCTAssertEqual(mock.lastTrackedEvent, "button_tapped")
    XCTAssertEqual(mock.trackEventCalls.first?.name, "button_tapped")
    XCTAssertEqual(mock.trackEventCalls.first?.parameters?["button_id"] as? String, "submit")
}
```

#### Способ 2 — Библиотека **Cuckoo** (очень популярна в 2026)

```swift
// Генерируем мок с помощью Cuckoo (нужен скрипт генерации)
mockAnalytics = mock(AnalyticsTracking.self)

// Настраиваем ожидание
stub(mockAnalytics) { stub in
    when(stub.trackEvent(name: "button_tapped", parameters: any())).thenDoNothing()
}

// Тест
viewModel.onButtonTap()

// Проверка
verify(mockAnalytics).trackEvent(name: "button_tapped", parameters: ["button_id": "submit"])
```

**Плюсы Cuckoo**:
- Автоматическая генерация моков  
- Читаемые `verify` и `when`  
- Поддержка [[async]]/[[await]] и [[actor]]

#### Способ 3 — **Mockingbird** / **SwiftMock** / **SwiftMockito** (альтернативы)

```swift
let mock = mock(AnalyticsTracking.self)
given(mock.trackEvent(name: "button_tapped", parameters: any())).willDoNothing()

viewModel.onButtonTap()

verify(mock.trackEvent(name: "button_tapped", parameters: ["button_id": "submit"])).wasCalled()
```

### 4. Лучшие практики использования Mock в Swift 2026

- **Используй протоколы** — моки работают только с протоколами  
- **Моки должны быть маленькими** — один мок = одна зависимость  
- **Проверяй только важные вызовы** — не проверяй каждый геттер/сеттер  
- **Не проверяй порядок вызовов**, если это не критично (хрупкие тесты)  
- **[[Async]]/[[await]]** — всегда используй `XCTestExpectation` или `await` в тестах  
- **[[@MainActor]]** — тесты ViewModel должны быть `@MainActor`  
- **Swift Testing** ([[Xcode]] 16+) — используй `@Test` + `#expect` + `await`  
- **Не мокай простые типы** ([[String]], [[Int]], [[Date]]) — используй реальные значения  
- **YAGNI в тестах** — не пиши мок на то, что не проверяешь

**Короткий девиз 2026**:
> «Mock — это не про имитацию всего подряд, а про проверку **конкретных взаимодействий** между объектами.  
> Хороший мок — маленький, читаемый и проверяет ровно то, что важно для теста.»
