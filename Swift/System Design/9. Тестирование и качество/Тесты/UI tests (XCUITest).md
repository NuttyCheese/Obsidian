#system_design
## Определение

**UI tests (XCUITest)** — это автоматизированные тесты пользовательского интерфейса в [[iOS]]-приложениях.

Цель: проверить, что **взаимодействие пользователя с приложением работает корректно**.

- Эмулируют действия пользователя: нажатия, свайпы, ввод текста.
    
- Проверяют визуальные элементы: наличие кнопок, лейблов, изображений.
    
- Обеспечивают **regression testing** при изменениях UI.
    

---

## 1. Принцип работы

1. XCUITest запускает приложение на симуляторе или реальном устройстве.
    
2. Скрипт выполняет **эмуляцию действий пользователя**.
    
3. Проверяет состояние UI через **assertions**.
    
4. Тест проходит, если UI соответствует ожидаемому поведению.
    

```text
Запуск приложения → Эмуляция действий → Проверка UI → Pass/Fail
```

---

## 2. Настройка XCUITest

1. В Xcode добавляем новый Target → **UI Testing Bundle**.
    
2. Импортируем **[[XCTest]]** в тестовый класс.
    
3. Создаём тестовые методы с префиксом `test`.
    

```swift
import XCTest

class MyAppUITests: XCTestCase {

    let app = XCUIApplication()
    
    override func setUp() {
        super.setUp()
        continueAfterFailure = false
        app.launch()
    }

    func testLoginFlow() {
        let usernameField = app.textFields["Username"]
        let passwordField = app.secureTextFields["Password"]
        let loginButton = app.buttons["Login"]

        usernameField.tap()
        usernameField.typeText("testuser")

        passwordField.tap()
        passwordField.typeText("password123")

        loginButton.tap()

        XCTAssertTrue(app.staticTexts["Welcome"].exists)
    }
}
```

---

## 3. Основные элементы XCUITest

|Элемент|Описание|
|---|---|
|`XCUIApplication`|Объект приложения для тестирования|
|`XCUIElement`|UI-элемент (кнопка, текстовое поле, таблица и т.д.)|
|`XCUIElementQuery`|Запрос для поиска элементов|
|Методы действий|`tap()`, `typeText()`, `swipeUp()`, `press(forDuration:)`|
|Assertions|`exists`, `isHittable`, `label` и др.|

---

## 4. Best Practices

1. **Использовать Accessibility Identifier**
    
    - Для стабильного поиска UI-элементов вместо `label` или `index`.
        

```swift
loginButton.accessibilityIdentifier = "login_button"
```

2. **Изоляция тестов**
    
    - Каждый тест должен запускаться независимо.
        
3. **Повторяемость**
    
    - Тесты должны работать на разных симуляторах и устройствах.
        
4. **Использование Launch Arguments / Environment**
    
    - Настройка приложения для тестов без изменения продакшн-данных.
        

```swift
app.launchArguments.append("--UITestMode")
```

5. **Ждать элементы**
    
    - Использовать `XCTWaiter` или `expectation(for:)` для асинхронных действий.
        

```swift
let exists = NSPredicate(format: "exists == true")
expectation(for: exists, evaluatedWith: loginButton, handler: nil)
waitForExpectations(timeout: 5)
```

---

## 5. Интеграция с [[CI]]/[[CD]]

- XCUITest полностью поддерживается [[Xcode]] Server, [[GitHub]] Actions, Bitrise и другими CI/CD сервисами.
    
- Позволяет **автоматически запускать UI-тесты при каждом пуше**.
    
- Совмещается с snapshot testing для проверки визуальных изменений.
    

---

## 6. Итог

- **XCUITest** позволяет автоматически тестировать поведение UI и пользовательские сценарии.
    
- Ключевые преимущества:
    
    - Проверка логики взаимодействия пользователя с UI.
        
    - Раннее выявление регрессий при изменениях интерфейса.
        
    - Возможность интеграции с CI/CD для постоянного контроля качества.
        
- Рекомендуется **использовать Accessibility Identifier, ожидания элементов и изоляцию тестов** для стабильности.
    

---
