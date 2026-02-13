### 1. Что такое UI-тесты и зачем они нужны в 2026 году

**UI-тесты** (XCUITest в [[iOS]]) — это **автоматизированные тесты**, которые:

- запускают **реальное приложение** в симуляторе / устройстве  
- имитируют **реальные действия пользователя** (тап, свайп, ввод текста, скролл)  
- проверяют **видимые изменения** на экране (существование элементов, текст, состояние, навигация)

**Главные цели UI-тестов в 2026**:

- Поймать **визуальные и навигационные регрессии**, которые unit-тесты не видят  
- Проверить **end-to-end пользовательские сценарии** (логин → покупка → профиль → выход)  
- Убедиться, что **UI соответствует требованиям** (доступность, локализация, тёмная/светлая тема)  
- Защитить **критические пути** приложения от случайных поломок при рефакторинге

**Актуальность 2026**:

- XCUITest — **основной и единственный официально поддерживаемый** инструмент Apple для UI-тестирования  
- С ростом [[SwiftUI]] и `@Observable` UI-тесты стали **ещё стабильнее**  
- В крупных проектах покрытие UI-тестами обычно **5–15%** критических сценариев (пирамида тестирования)

### 2. Преимущества UI-тестов (реальные плюсы 2026)

1. **Проверяют реальное приложение**  
   → Ловят баги, которые unit-тесты пропускают (анимации, layout, доступность, локализация)

2. **Ближе всего к пользовательскому опыту**  
   → Тесты имитируют реальные действия → дают уверенность, что пользователь ничего не сломает

3. **Защищают критические пути**  
   → Логин → покупка → профиль → выход — если это сломалось, бизнес теряет деньги

4. **Хорошо работают в [[CI]]/[[CD]]**  
   → [[GitHub]] Actions / [[Xcode]] Cloud / Bitrise запускают UI-тесты на реальных устройствах

5. **Поддержка тёмной/светлой темы и локализации**  
   → Легко проверять разные trait collection и language settings

### 3. Недостатки и риски UI-тестов (честные минусы 2026)

1. **Очень медленные**  
   → 1 тест = 10–120 секунд → весь сьют может идти 10–60 минут

2. **Хрупкие**  
   → Изменение текста кнопки / accessibility identifier / анимации → тест падает  
   → Переезд на новый дизайн → нужно переписывать 50–70% тестов

3. **Дорогие в поддержке**  
   → Средний UI-тест живёт 3–6 месяцев до первого редизайна  
   → Команда тратит 30–50% времени на поддержку тестов

4. **Плохо отлаживаются**  
   → Падает на скриншоте или логе → поиск причины занимает 10–60 минут

5. **Не ловят внутреннюю логику**  
   → Если ViewModel вернул неправильные данные, но UI их красиво отобразил — тест пройдёт

6. **Зависимость от симулятора/устройства**  
   → Разные версии [[iOS]], разные устройства → flaky-тесты (нестабильные)

### 4. Самые популярные шаблоны UI-тестов в Swift 2026

#### Шаблон 1 — Базовый тест логин-экрана (XCUITest)

```swift
import XCTest

final class LoginUITests: XCTestCase {
    
    var app: XCUIApplication!
    
    override func setUpWithError() throws {
        continueAfterFailure = false
        app = XCUIApplication()
        app.launchArguments += ["UI-Testing"]
        app.launch()
    }
    
    func testSuccessfulLogin() throws {
        // Arrange
        let usernameField = app.textFields["usernameTextField"]
        let passwordField = app.secureTextFields["passwordSecureField"]
        let loginButton = app.buttons["loginButton"]
        
        // Act
        usernameField.tap()
        usernameField.typeText("test@example.com")
        
        passwordField.tap()
        passwordField.typeText("password123")
        
        loginButton.tap()
        
        // Assert
        let homeScreen = app.otherElements["homeScreenView"]
        XCTAssertTrue(homeScreen.waitForExistence(timeout: 5))
    }
    
    func testInvalidCredentialsShowsError() throws {
        // Arrange + Act
        app.textFields["usernameTextField"].tap()
        app.textFields["usernameTextField"].typeText("wrong@example.com")
        
        app.secureTextFields["passwordSecureField"].tap()
        app.secureTextFields["passwordSecureField"].typeText("wrong")
        
        app.buttons["loginButton"].tap()
        
        // Assert
        let errorAlert = app.alerts["errorAlert"]
        XCTAssertTrue(errorAlert.waitForExistence(timeout: 3))
        XCTAssertTrue(errorAlert.staticTexts["Invalid credentials"].exists)
    }
}
```

#### Шаблон 2 — Тест с разными состояниями (loading, [[error]], empty)

```swift
func testLoadingState() throws {
    // Запускаем приложение с флагом loading
    app.launchArguments += ["UI-Testing-Loading"]
    app.launch()
    
    let loadingIndicator = app.activityIndicators["loadingIndicator"]
    XCTAssertTrue(loadingIndicator.exists)
}

func testErrorState() throws {
    app.launchArguments += ["UI-Testing-Error"]
    app.launch()
    
    let errorLabel = app.staticTexts["errorMessage"]
    XCTAssertTrue(errorLabel.waitForExistence(timeout: 5))
    XCTAssertEqual(errorLabel.label, "Failed to load data")
}
```

#### Шаблон 3 — Тест SwiftUI View (гибридный подход)

```swift
func testProfileViewDisplaysUserName() throws {
    let app = XCUIApplication()
    app.launchArguments += ["UI-Testing-Profile"]
    app.launch()
    
    let nameLabel = app.staticTexts["profileNameLabel"]
    XCTAssertTrue(nameLabel.waitForExistence(timeout: 5))
    XCTAssertEqual(nameLabel.label, "Alex Smith")
}
```

### 5. Лучшие практики UI-тестирования в Swift 2026

- **Accessibility identifiers** — **обязательны** для всех кнопок, полей, лейблов  
  → `accessibilityIdentifier = "loginButton"`

- **Launch arguments** — используйте для подмены состояний (loading, error, [[mock]] data)  
  → `app.launchArguments += ["UI-Testing-Error"]`

- **waitForExistence(timeout:)** — **всегда** ждите элементы, не полагайтесь на мгновенное появление  
  → timeout: 3–8 секунд

- **Тёмная/светлая тема** — тестируйте оба режима  
  → `app.overrideUserInterfaceStyle = .dark`

- **Локализация** — хотя бы en + ru (или ваши основные)  
  → `app.launchArguments += ["-AppleLanguages", "(ru)"]`

- **Разные устройства** — добавляйте тесты на iPhone SE, iPhone 15 Pro Max, iPad  
  → Используйте разные симуляторы в CI

- **Flaky-тесты** — минимизируйте с помощью `waitForExistence` и стабильных identifiers  
- **CI/CD** — запускайте UI-тесты **только на release-ветках** или nightly (они медленные)  
- **Swift 6 strict concurrency** — тесты должны быть `@MainActor` или использовать `await MainActor.run`

### 6. Когда UI-тесты действительно окупаются (2026)

| Тип экрана / сценария                               | Стоит ли писать UI-тесты? | Рекомендуемое покрытие |
|-----------------------------------------------------|----------------------------|-------------------------|
| Критические пользовательские пути (логин → покупка) | Да, обязательно            | 80–100%                 |
| Основные экраны приложения (профиль, каталог, корзина) | Да                         | 60–90%                  |
| Экраны с множеством состояний (loading, error, empty) | Да                         | 70–100%                 |
| Редко используемые экраны / настройки               | Нет / минимально           | 0–30%                   |
| Временные прототипы / эксперименты                  | Нет                        | 0%                      |

### 7. Итог: UI-тесты — мощный, но дорогой инструмент

- **Пиши UI-тесты** на **критические пути** и **ключевые экраны**  
- **Не пиши** на каждый лейбл и кнопку — это over-testing  
- **Цель** — **5–15% покрытия** критических сценариев (пирамида тестирования)  
- **Лучший баланс** — быстрые unit-тесты (70–85%) + снапшоты (20–30%) + UI-тесты (5–15%)

**Короткий девиз 2026**:
> «UI-тесты — это страховка от самых дорогих багов: когда пользователь не может войти, купить или увидеть свой профиль.  
> Но они медленные и хрупкие — используй их только там, где цена ошибки максимальна.»
