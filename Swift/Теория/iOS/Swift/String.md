**`String`** — это **основной тип** в [[Swift]] для работы с **текстом**.  
Это **структура** ([[struct]]), **[[value type]]**, полностью **[[Unicode]]-совместимая** и оптимизированная для современных приложений.

В 2025–2026 годах `String` остаётся одним из самых часто используемых типов, и его API продолжает улучшаться (особенно в Swift 5.5+ и Swift 6).

### 1. Ключевые характеристики String (актуально 2026)

| Характеристика         | Значение / Особенность                              | Почему это важно                                  |
| ---------------------- | --------------------------------------------------- | ------------------------------------------------- |
| Тип                    | `struct` (value type)                               | Копируется при присваивании ([[Copy-on-Write]])   |
| Кодировка              | UTF-8 (внутренне)                                   | Экономия памяти + быстрая работа с emoji          |
| Элемент                | `Character` (графемный кластер)                     | Корректно считает emoji и комбинированные символы |
| Индексация             | `String.Index` (не Int!)                            | Безопасный доступ, защита от ошибок               |
| Immutable по умолчанию | `let str = "text"` — нельзя изменить                | Безопасность + оптимизация                        |
| Mutable через `var`    | `var str = "text"` → можно `+=`, `append`, `remove` | Гибкость                                          |
| Интерполяция           | `"Hello, \(name)"`                                  | Самый читаемый способ                             |
| Многострочные строки   | `""" ... """`                                       | Удобно для [[JSON]], HTML, SQL                    |
| Поддержка Unicode      | Полная (emoji, RTL, комбинированные символы)        | Работает с любым языком                           |

### 2. Самые важные свойства и методы String (топ-2026)

| Свойство / Метод                  | Что возвращает / делает                                      | Пример 2026 |
|-----------------------------------|--------------------------------------------------------------|-------------|
| `count`                           | Количество **символов** (графем)                             | `"👨‍💻".count` → 1 |
| `isEmpty`                         | Пустая ли строка                                             | `text.isEmpty` |
| `startIndex`, `endIndex`          | Начало и конец строки                                        | `text.startIndex` |
| `index(_:offsetBy:)`              | Получение индекса со смещением                               | `text.index(text.startIndex, offsetBy: 5)` |
| `prefix(_:)` / `suffix(_:)`       | Первые/последние N символов                                  | `text.prefix(5)` → Substring |
| `uppercased()`, `lowercased()`    | Верхний/нижний регистр                                       | `text.uppercased()` |
| `trimmingCharacters(in:)`         | Удаление пробелов и символов                                 | `text.trimmingCharacters(in: .whitespacesAndNewlines)` |
| `contains(_:)`                    | Содержит ли подстроку / символ                               | `text.contains("Swift")` |
| `replacingOccurrences(of:with:)`  | Замена подстрок                                              | `text.replacingOccurrences(of: "old", with: "new")` |
| `split(separator:maxSplits:)`     | Разбиение по разделителю                                     | `text.split(separator: " ")` → [Substring] |
| `appending(_:)` / `+=`            | Добавление строки                                            | `var s = "Hello"; s += " World"` |
| `joined(separator:)`              | Объединение коллекции строк                                  | `["a", "b"].joined(separator: "-")` → "a-b" |

### 3. Самые популярные паттерны String в 2026

#### 3.1 Интерполяция и многострочные строки (самый частый)

```swift
let name = "Алекс"
let age = 28

let greeting = """
Привет, \(name)!
Тебе \(age) лет.
"""
```

#### 3.2 Безопасное извлечение подстроки (индексы)

```swift
let text = "Hello, Swift!"
let start = text.index(text.startIndex, offsetBy: 7)
let end = text.index(start, offsetBy: 5)
let word = text[start..<end]     // "Swift"
let substring = String(word)     // если нужен полноценный String
```

#### 3.3 Обработка пользовательского ввода (очень часто)

```swift
var input = readLine()?.trimmingCharacters(in: .whitespacesAndNewlines) ?? ""
input = input.lowercased()

if input.isEmpty {
    print("Пустой ввод")
} else if input.hasPrefix("hello") {
    print("Привет!")
}
```

#### 3.4 Форматирование текста (UI, логи, API)

```swift
let price = 1299.99
let formatted = String(format: "%.2f ₽", price)  // "1299.99 ₽"

let attributed = NSMutableAttributedString(string: "Скидка 20%")
attributed.addAttribute(.foregroundColor, value: UIColor.red, range: NSRange(location: 7, length: 3))
```

#### 3.5 Работа с emoji и Unicode

```swift
let emojiText = "👨‍💻 Swift ❤️"
print(emojiText.count)           // 4 (графемных кластера)
print(emojiText.unicodeScalars.count)  // больше (байты)

for char in emojiText {
    print(char)  // 👨‍💻, " ", S, w, i, f, t, " ", ❤️
}
```

### 4. Лучшие практики String в Swift 2026

- **Используй интерполяцию** (`\(…)`) вместо `+` — читаемее и быстрее  
- **Для больших строк** — используй многострочные `"""`  
- **Для ввода** — всегда `trimmingCharacters(in: .whitespacesAndNewlines)` + `.lowercased()`  
- **Для индексации** — **никогда** не используй `Int` напрямую — только `String.Index`  
- **Для производительности** — избегай многократного `+=` в цикле — лучше `String` + `append` или `join`  
- **Для форматирования** — `String(format:)` или `NumberFormatter` / `DateFormatter`  
- **В SwiftUI** — часто комбинируй с `AttributedString` (iOS 15+)  
- **Swift 6 strict concurrency** — `String` полностью `Sendable`  
- **Документируйте** — пиши комментарий «String — отформатированное сообщение с именем пользователя»

**Короткий девиз 2026**:
> `String` — это **текстовая суперсила** Swift: Unicode, интерполяция, безопасные срезы, emoji без проблем.  
> В 2026 году:  
> - интерполяция и многострочные строки — основной способ  
> - `String.Index` — только через `index(_:offsetBy:)`  
> - `trim`, `contains`, `replacingOccurrences` — для очистки и поиска  
> - `+=` в цикле — антипаттерн, используй `join` или builder  
> Это **самый часто используемый** тип после `Int` и `Bool`.
