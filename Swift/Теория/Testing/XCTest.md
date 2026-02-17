**XCTest** — это **встроенный** фреймворк Apple для модульного, интеграционного и производительного тестирования в [[Swift]] и [[Objective-C]].

С 2013 года (Xcode 5) он является **стандартом де-факто** для [[iOS]]/macOS/watchOS/tvOS-разработки.

**Ключевые изменения к 2026 году**:

- **Swift Testing** (новый фреймворк с Xcode 16, 2024) **не заменил** XCTest, а **дополнил** его  
- XCTest остаётся **основным** для большинства проектов  
- Swift Testing рекомендуется для **новых проектов** и **чистого Swift-кода**  
- Оба фреймворка отлично работают вместе в одном проекте

### 2. Основные возможности XCTest (2026 актуальные)

| Возможность                        | Описание                                                                 | Статус в 2026 |
|------------------------------------|--------------------------------------------------------------------------|---------------|
| `XCTestCase`                       | Базовый класс для тестов                                                 | ★★★★★         |
| `XCTAssert*`                       | Классические утверждения (Equal, True, Nil, Throws и т.д.)              | ★★★★★         |
| `async throws` тесты               | Полная поддержка async/await без `XCTestExpectation`                     | ★★★★★         |
| `@MainActor` в тестах              | Обязателен для ViewModel / UI-related тестов                             | ★★★★★         |
| `measure {}`                       | Тестирование производительности                                          | ★★★★☆         |
| `setUp()` / `tearDown()`           | Подготовка и очистка перед/после каждого теста                           | ★★★★★         |
| `XCTAttachment`                    | Добавление файлов/скриншотов в отчёт об ошибке                           | ★★★★☆         |
| `XCTContext`                       | Группировка тестов и добавление активности (run activity)               | ★★★★☆         |

### 3. Самые популярные и рекомендуемые шаблоны тестов в XCTest 2026

#### Шаблон 1 — Простой синхронный тест

```swift
import XCTest
@testable import MyMathLib

final class MathTests: XCTestCase {
    
    func testAdditionOfPositiveNumbers() {
        // Arrange
        let a = 5
        let b = 7
        
        // Act
        let result = Math.add(a, b)
        
        // Assert
        XCTAssertEqual(result, 12, "5 + 7 должно быть 12")
    }
}
```

#### Шаблон 2 — Асинхронный тест (самый частый в 2026)

```swift
final class UserServiceTests: XCTestCase {
    
    @MainActor
    func testFetchUserSuccess() async throws {
        // Arrange
        let stub = UserServiceStub(users: [User(id: 1, name: "Alice")])
        let service = UserService(repository: stub)
        
        // Act
        let users = try await service.fetchUsers()
        
        // Assert
        XCTAssertEqual(users.count, 1)
        XCTAssertEqual(users.first?.name, "Alice")
    }
    
    func testFetchUserFailure() async throws {
        // Arrange
        let stub = UserServiceStub(shouldThrow: true)
        let service = UserService(repository: stub)
        
        // Act + Assert
        await XCTAssertThrowsError(try await service.fetchUsers()) { error in
            XCTAssertEqual(error as? NetworkError, .noConnection)
        }
    }
}
```

#### Шаблон 3 — Тест производительности (measure)

```swift
func testHeavyComputationPerformance() {
    let input = Array(0..<1_000_000)
    
    measure {
        // Act — измеряем только эту часть
        _ = input.sorted()
    }
}
```

**Важно**: `measure` запускает блок много раз → убирайте из него всё лишнее (Arrange/Assert вне блока).

#### Шаблон 4 — Тест с XCTAttachment (скриншоты при падении)

```swift
func testViewControllerLayout() {
    let vc = ProfileViewController()
    vc.loadViewIfNeeded()
    
    let attachment = XCTAttachment(screenshotOf: vc.view)
    attachment.name = "ProfileViewController - initial state"
    add(attachment)
    
    FBSnapshotVerifyView(vc.view) // если используете iOSSnapshotTestCase
}
```

### 4. Современный подход 2026 — XCTest + Swift Testing вместе

```swift
import XCTest
import Testing

final class MathTests: XCTestCase {
    
    @Test("Addition works correctly")
    func addition() {
        #expect(Math.add(2, 3) == 5)
        #expect(Math.add(-1, 1) == 0)
    }
    
    @Test("Division by zero throws")
    func divisionByZero() throws {
        #expect(throws: MathError.divisionByZero) {
            try Math.divide(10, by: 0)
        }
    }
}
```

**Swift Testing** (Xcode 16+):
- `@Test`, `@Suite`, `#expect`, `#require`  
- Лучшая читаемость  
- Полная поддержка [[async]]/[[await]]  
- Можно использовать **вместе** с XCTest в одном проекте

### 5. Лучшие практики unit-тестирования в Swift 2026

- **Один тест — одна идея** (1 Act + 1–3 Assert)  
- **Тестируй поведение, а не реализацию**  
- **Моки/стабы через протоколы** — самый надёжный способ  
- **Async-тесты** — всегда `async throws` + `await`  
- **[[@MainActor]]** — обязателен для ViewModel/UI-тестов  
- **Coverage** — 70–85% бизнес-логики (не стремись к 100%)  
- **Не тестируй очевидное** ([[YAGNI]] в тестах)  
- **Swift Testing** — переходи на него для новых проектов  
- **XCTAttachment** — добавляй скриншоты/логи при падении  
- **Swift 6 strict concurrency** — тесты помогают ловить ошибки изоляции

### 6. Итог — когда и сколько unit-тестов писать

- **Пиши unit-тесты** на всю **бизнес-логику**, **ViewModel**, **UseCase**, **валидацию**, **маппинг**  
- **Не пиши** на простые модели, UI без состояния, очевидные геттеры/сеттеры  
- **Цель** — **70–85% покрытия** ключевой логики  
- **Лучший баланс** — быстрые, читаемые, устойчивые тесты, которые дают уверенность при рефакторинге

**Короткий девиз 2026**:
> «Unit-тесты — это страховка от регрессий и гарантия, что завтрашний рефакторинг не сломает сегодняшний функционал.  
> Но если тесты занимают больше времени, чем сам код — ты переусердствовал.»
