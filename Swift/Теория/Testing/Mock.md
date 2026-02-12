**Mock (Имитация):**

- Мок - это объект, который имитирует поведение реального объекта, но также способен записывать вызовы и проверять их в тестах.
- Его цель - проверить, что определенные методы были вызваны с определенными аргументами и в определенном порядке.
- Обычно используется для проверки взаимодействия между объектами и их методами в тестах.
- Пример: создание мока для сервиса уведомлений, чтобы убедиться, что метод отправки уведомлений был вызван с правильными параметрами.
Пример использования мока (Mock):

Предположим, у нас есть класс `AnalyticsManager`, который отправляет аналитические события, и мы хотим проверить, что метод `trackEvent()` был вызван с правильными параметрами. Мы создадим мок `AnalyticsManagerMock`, чтобы проверить это в тесте.
```swift
protocol AnalyticsManager {
    func trackEvent(name: String, parameters: [String: Any])
}

class EventLogger {
    private let analyticsManager: AnalyticsManager
    
    init(analyticsManager: AnalyticsManager) {
        self.analyticsManager = analyticsManager
    }
    
    func logEvent() {
        analyticsManager.trackEvent(name: "ButtonPressed", parameters: ["buttonName": "Submit"])
    }
}

// Мок AnalyticsManager для проверки вызова метода trackEvent
class AnalyticsManagerMock: AnalyticsManager {
    var trackEventCalled = false
    var trackEventName: String?
    var trackEventParameters: [String: Any]?
    
    func trackEvent(name: String, parameters: [String: Any]) {
        trackEventCalled = true
        trackEventName = name
        trackEventParameters = parameters
    }
}

// Тестирование EventLogger с использованием мока
func testEventLogger() {
    let mock = AnalyticsManagerMock()
    let eventLogger = EventLogger(analyticsManager: mock)
    
    eventLogger.logEvent()
    
    XCTAssertTrue(mock.trackEventCalled)
    XCTAssertEqual(mock.trackEventName, "ButtonPressed")
    XCTAssertEqual(mock.trackEventParameters?["buttonName"] as? String, "Submit")
}

