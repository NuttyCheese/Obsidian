**Unicode** — это универсальный стандарт кодирования символов, который позволяет компьютерам корректно представлять и обрабатывать текст **на любом языке мира**, включая редкие письменности, эмодзи, математические символы и специальные знаки.

В Swift поддержка Unicode реализована **очень глубоко** и является одной из сильнейших сторон языка.

### 1. Основные уровни Unicode в [[Swift]] (как это устроено внутри)

| Уровень                 | Тип в Swift              | Что представляет                                           | Кол-во кодовых точек на символ | Пример (флаг России 🇷🇺)      |
| ----------------------- | ------------------------ | ---------------------------------------------------------- | ------------------------------ | ------------------------------ |
| Unicode Scalar          | `Unicode.Scalar`         | Одна кодовая точка (code point)                            | 1                              | две scalars: U+1F1F7 + U+1F1FA |
| Grapheme Cluster        | [[Character]]            | Один видимый символ (может состоять из нескольких scalars) | 1+                             | 1 `Character`                  |
| String                  | [[String]]               | Последовательность `Character`                             | —                              | 1 `String`                     |
| UTF-8 / UTF-16 / UTF-32 | внутренние представления | Как строка хранится в памяти                               | —                              | UTF-8 (8 байт для 🇷🇺)        |

**Ключевой момент 2026 года**:  
Swift всегда работает с текстом на уровне **grapheme clusters** (`Character`), а не на уровне отдельных кодовых точек.  
Поэтому `"🇷🇺".count == 1`, а не 2.

### 2. Самые важные и часто используемые возможности Unicode в Swift

#### 2.1 Подсчёт символов (самый частый вопрос на собеседованиях)

```swift
let text = "café 👨‍💻 🇷🇺"
print(text.count)                    // 6 (графем / видимых символов)
print(text.unicodeScalars.count)     // 10 (отдельные кодовые точки)
print(text.utf8.count)               // 22 байта в UTF-8
print(text.utf16.count)              // 12 единиц в UTF-16
```

#### 2.2 Корректная работа с эмодзи и комбинированными символами

```swift
let skinTones = "👨🏻‍💻👨🏼‍💻👨🏽‍💻👨🏾‍💻👨🏿‍💻"
print(skinTones.count)               // 5 (каждый — отдельный символ)

let eWithAcute = "é"                 // один символ
let ePlusAcute = "e\u{0301}"         // тоже один символ
print(eWithAcute == ePlusAcute)      // true (Unicode-нормализация)
```

#### 2.3 Безопасные срезы строк (самый частый баг у новичков)

```swift
let greeting = "Привет, мир!"
let start = greeting.index(greeting.startIndex, offsetBy: 7)
let end   = greeting.index(start, offsetBy: 3)

let substring = greeting[start..<end]   // "мир"
print(substring)

// НЕ ДЕЛАЙТЕ ТАК:
let wrong = greeting[7..<10]            // Ошибка компиляции — Int не String.Index
```

#### 2.4 Поиск и замена с учётом Unicode

```swift
let text = "Приве́т, ми́р! 😊"
let normalized = text.folding(options: .diacriticInsensitive, locale: .current)
print(normalized)  // "Привет, мир! 😊" (без акутов)

let hasSmile = text.contains("😊")          // true
let hasRussian = text.range(of: "мир") != nil // true
```

### 3. Лучшие практики работы с Unicode в Swift 2026

- **Всегда** используйте `Character` и `.count` для подсчёта видимых символов  
- **Никогда** не используйте `.utf8.count` / `.utf16.count` для отображения длины текста пользователю  
- **Для сравнения без учёта регистра/акцентов** — `.folding(options: .caseInsensitive + .diacriticInsensitive)` или `.localizedStandardCompare`  
- **Для нормализации** — `.precomposedStringWithCanonicalMapping` / `.decomposedStringWithCanonicalMapping`  
- **Для безопасного среза** — всегда через `index(_:offsetBy:)` или `prefix(_:)` / `suffix(_:)`  
- **В SwiftUI** — `Text` автоматически корректно отображает все Unicode-символы  
- **Документируйте** — пишите комментарий «строка в NFC-нормализации (для корректного сравнения)»

**Короткий итог 2026**:
> Swift — один из лучших языков по поддержке Unicode «из коробки».  
> Главные правила:  
> - `count` — это количество видимых символов (`Character`), а не байтов  
> - эмодзи и комбинированные символы обрабатываются корректно  
> - индексация только через `String.Index` — никогда не используйте `Int`  
> - для сравнения и поиска — используйте `.folding`, `.localizedStandardCompare`  
> - срезы — через `prefix`, `suffix`, `index(_:offsetBy:)`  
