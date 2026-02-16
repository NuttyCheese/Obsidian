**Decimal** в Swift — это структура из **Foundation**, предназначенная для **точных десятичных вычислений** с фиксированной точкой. Это **единственный рекомендуемый** тип в Swift для работы с деньгами, финансами, налогами, ценами и любыми расчётами, где **потеря точности недопустима**.

В 2026 году Decimal остаётся **золотым стандартом** для финансовых приложений, бухгалтерии, e-commerce и крипто-приложений.

### Почему именно Decimal (а не Double / Float)

| Тип          | Точность при вычислениях | Подходит для денег? | Пример проблемы с Double | Рекомендация 2026 |
|--------------|--------------------------|----------------------|---------------------------|-------------------|
| **Double**   | Приближённая (бинарная с плавающей точкой) | **Нет**              | 0.1 + 0.2 = 0.30000000000000004 | Никогда для финансов |
| **Float**    | Ещё хуже, чем Double     | **Нет**              | Ещё больше ошибок округления   | Только графика/ML |
| **Decimal**  | Точная (десятичная с плавающей точкой) | **Да**               | 0.1 + 0.2 = 0.3 (точно)       | **Всегда** для денег |

### Основные свойства и возможности Decimal

```swift
let price = Decimal(string: "19.99")!          // из строки — самый безопасный способ
let taxRate = Decimal("0.20")!
let quantity = Decimal(3)

let subtotal = price * quantity                 // 59.97
let tax = subtotal * taxRate                    // 11.994
let total = subtotal + tax                      // 71.964

// Округление (самое важное для финансов)
let roundedTotal = total.rounded(.plain(scale: 2))  // 71.96
// или
let banker'sRounded = NSDecimalNumber(decimal: total)
    .rounding(accordingToBehavior: NSDecimalNumberHandler(roundingMode: .bankers, scale: 2, raiseOnExactness: false, raiseOnOverflow: false, raiseOnUnderflow: false, raiseOnDivideByZero: false))
    .decimalValue  // 71.96 (Banker's rounding)
```

### Самые популярные и рекомендуемые паттерны 2026 года

#### 1. Парсинг из строки (самый безопасный способ)

```swift
extension Decimal {
    init?(safeString: String) {
        guard let decimal = Decimal(string: safeString) else { return nil }
        self = decimal
    }
}

// Использование
guard let price = Decimal(safeString: "19.99") else {
    // обработка ошибки
}
```

#### 2. Форматирование для отображения (валюта)

```swift
let formatter = NumberFormatter()
formatter.numberStyle = .currency
formatter.currencyCode = "USD"
formatter.locale = Locale(identifier: "en_US")

let price = Decimal(string: "1234.567")!
let formatted = formatter.string(from: price as NSDecimalNumber) 
// → "$1,234.57"
```

#### 3. Точный расчёт скидки / налога

```swift
func calculateTotal(price: Decimal, discountPercent: Decimal, taxPercent: Decimal) -> Decimal {
    let discount = price * (discountPercent / 100)
    let discounted = price - discount
    let tax = discounted * (taxPercent / 100)
    return (discounted + tax).rounded(.plain(scale: 2))
}

let total = calculateTotal(price: 99.99, discountPercent: 15, taxPercent: 20)
// → 95.988 → 95.99 после округления
```

#### 4. Decimal в Codable / JSON

```swift
struct Product: Codable {
    let name: String
    let price: Decimal
    
    enum CodingKeys: String, CodingKey {
        case name, price
    }
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        name = try container.decode(String.self, forKey: .name)
        
        // Самый безопасный способ декодирования Decimal из JSON
        if let priceString = try? container.decode(String.self, forKey: .price) {
            guard let decimal = Decimal(string: priceString) else {
                throw DecodingError.dataCorruptedError(forKey: .price, in: container, debugDescription: "Invalid decimal string")
            }
            price = decimal
        } else if let priceDouble = try? container.decode(Double.self, forKey: .price) {
            price = Decimal(priceDouble)  // но это менее точно
        } else {
            throw DecodingError.keyNotFound(.price, .init(codingPath: [], debugDescription: "Missing price"))
        }
    }
}
```

### Лучшие практики Decimal в Swift 2026

- **Всегда создавай Decimal из строки** (`Decimal(string:)`) — это самый точный способ  
- **Никогда не используй Double → Decimal** — потеря точности неизбежна  
- **Округляй явно** — `.rounded(.plain(scale: 2))` или `NSDecimalNumberHandler` с нужным режимом (`.bankers`, `.plain`, `.up` и т.д.)  
- **Используй NSDecimalNumberHandler** — для полного контроля над rounding mode и scale  
- **Swift 6 strict concurrency** — Decimal полностью Sendable — безопасен для actor и Task  
- **Codable** — декодируй из строки, а не из Double  
- **Тестирование** — проверяй с помощью XCTAssertEqual с точностью 0 (Decimal сравнивается точно)  
- **Документируйте** — пиши комментарий «Decimal — точные финансовые расчёты, округление до 2 знаков»

**Короткий девиз 2026**:
> «Decimal — это когда тебе нужны **точные** деньги, цены, налоги, скидки без потери копеек и без 0.30000000000000004.  
> В 2026 году это **единственный** правильный выбор для всех финансовых расчётов в Swift.  
> Никогда не используй Double/Float для денег — только Decimal.»

Удачи с точными и надёжными финансовыми расчётами в Swift! 💰