#tests #automation 
## 📘 Определение

**XCTest** — это **фреймворк для модульного тестирования в [[Swift]] и [[Objective-C]]**.  
Позволяет создавать **юнит-тесты, производить проверки и измерять производительность кода**.  
Относится к **[[Xcode]] / Testing**.

---

## 🔹 Примеры кода

### 1. Простейший тест

```swift
import XCTest

class MathTests: XCTestCase {
    func testAddition() {
        let sum = 2 + 3
        XCTAssertEqual(sum, 5) // проверяем, что результат равен 5
    }
}
```

---

### 2. Проверка условия с XCTAssertTrue

```swift
func testIsEven() {
    let number = 4
    XCTAssertTrue(number % 2 == 0, "Число должно быть четным")
}
```

---

### 3. Проверка на [[nil]]

```swift
func testOptional() {
    let value: String? = "Hello"
    XCTAssertNotNil(value, "Значение не должно быть nil")
}
```

---

### 4. Проверка с XCTAssertThrowsError

```swift
enum TestError: Error { case fail }

func testThrows() {
    func willThrow() throws {
        throw TestError.fail
    }
    
    XCTAssertThrowsError(try willThrow()) { error in
        XCTAssertEqual(error as? TestError, TestError.fail)
    }
}
```

---

### 5. Производительность с measure

```swift
func testPerformanceExample() {
    measure {
        _ = (1...1000).map { $0 * 2 }
    }
}
```
