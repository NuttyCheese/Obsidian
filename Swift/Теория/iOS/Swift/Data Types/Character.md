**`Character`** в [[Swift]] — это тип, который представляет **один видимый графический символ** (grapheme cluster), а не один Unicode-скаляр или байт.

Это ключевое отличие Swift от многих других языков: строка в Swift — это не массив байтов и не массив скаляров, а последовательность именно `Character` (расширенных графемных кластеров).

### 1. Почему Character — это не просто Unicode.Scalar

| Тип              | Что хранит                             | Примеры символов                    | Кол-во кодовых точек | Видимый результат          |
| ---------------- | -------------------------------------- | ----------------------------------- | -------------------- | -------------------------- |
| `Unicode.Scalar` | Одна кодовая точка Unicode (21 бит)    | "e", "́", "😀", "🇺🇦"              | 1                    | Не всегда видимый символ   |
| `Character`      | Один **расширенный графемный кластер** | "e", "é", "😀", "🇺🇦", "å", "1️⃣" | 1–несколько          | Всегда один видимый символ |
| [[String]]       | Коллекция `Character`                  | "café", "Swift❤️"                   | Любое                | Полный текст               |

Примеры, где `Character` ≠ `Unicode.Scalar`:

```swift
let eAcute: Character = "é"           // 1 Character
print(eAcute.unicodeScalars.count)    // 2 (e + ◌́)

let emoji: Character = "😀"           // 1 Character
print(emoji.unicodeScalars.count)     // 1

let flag: Character = "🇺🇦"           // 1 Character (флаг Украины)
print(flag.unicodeScalars.count)      // 2 (региональные индикаторы U + R)

let combined: Character = "å"        // a + кольцо сверху
print(combined.unicodeScalars.count)  // 2
```

### 2. Как правильно работать с Character (лучшие практики 2026)

#### Получение Character из строки

```swift
let text = "café ❤️ 🇺🇦"
for char in text {
    print(char, "→", char.unicodeScalars.map { $0.value }) 
    // c → [99]
    // a → [97]
    // f → [102]
    // é → [101, 769]
    //   → [32]
    // ❤️ → [10084, 65039]
    //   → [32]
    // 🇺 → [127482]
    // 🇦 → [127462]
}
```

#### Подсчёт символов (самый частый вопрос)

```swift
let text = "café ❤️ 🇺🇦"
print(text.count)               // 7 (количество Character)
print(text.unicodeScalars.count) // 10 (количество скаляров)
```

**Правило 2026**:  
Всегда используйте `.count` для подсчёта видимых символов — это именно количество `Character`.

#### Сравнение и сортировка

```swift
let a: Character = "a"
let eAcute: Character = "é"
let emoji: Character = "😀"

print(a < eAcute)      // true (лексикографически)
print(eAcute < emoji)  // true
print("é" == "é")     // true — Swift нормализует и сравнивает графемы
```

#### Работа с эмодзи и модификаторами

```swift
let family: Character = "👨‍👩‍👧‍👦"  // семья: мужчина + женщина + девочка + мальчик
print(family.unicodeScalars.count)   // 7 (4 человека + 3 zero-width joiner)

let skinTone: Character = "👨🏾"     // мужчина с тёмным тоном кожи
print(skinTone.unicodeScalars.count) // 2 (👨 + модификатор тона)
```

### 3. Полезные расширения Character (очень популярны в 2026)

```swift
extension Character {
    /// Является ли символ эмодзи
    var isEmoji: Bool {
        unicodeScalars.contains { $0.properties.isEmojiPresentationDefault }
    }
    
    /// Является ли символ буквой или цифрой
    var isAlphanumeric: Bool {
        unicodeScalars.allSatisfy { $0.properties.isAlphabetic || $0.properties.isDecimalDigit }
    }
    
    /// Возвращает строку из одного символа (удобно для API)
    var string: String { String(self) }
}

// Примеры
let heart = "❤️"
print(heart.isEmoji)          // true
print("a".isAlphanumeric)     // true
print("1".isAlphanumeric)     // true
print("!".isAlphanumeric)     // false
```

### 4. Лучшие практики Character в Swift 2026

- **Всегда итерируйте строку через `for char in string`** — это правильно обрабатывает графемы  
- **Не итерируйте `unicodeScalars` или `utf8` вручную**, если вам нужны видимые символы  
- **Для подсчёта длины** — используйте `string.count`, а не `unicodeScalars.count`  
- **Для работы с эмодзи** — проверяйте `unicodeScalars` свойства (`isEmoji`, `isEmojiPresentationDefault`)  
- **В [[SwiftUI]] / [[UIKit]]** — `Character` безопасен для отображения и ввода текста  
- **Swift 6 strict concurrency** — `Character` полностью `Sendable` и безопасен  
- **Документируйте** — пиши комментарий «Character — один видимый символ (графемный кластер)»

**Короткий девиз 2026**:
> `Character` — это **один видимый символ**, даже если он состоит из нескольких скаляров (диакритика, эмодзи, флаги, zero-width joiner).  
> В 2026 году итерируй строки именно по `Character`, считай длину через `.count`, и помни: Swift — один из немногих языков, который правильно обрабатывает эмодзи и составные символы из коробки.
