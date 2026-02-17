**Unicode** — это **стандарт кодирования символов**, который позволяет представлять **текст на любом языке мира**.

- Каждый символ (буква, цифра, знак) имеет уникальный **код (code point)**.
    
- Unicode поддерживает тысячи символов: латиницу, кириллицу, иероглифы, эмодзи и специальные знаки.
    

В [[Swift]] все строки ([[String]]) **используют Unicode**, что позволяет работать с многоязычным текстом корректно.

---

## 2. Unicode в Swift

В Swift `String` и `Character` полностью поддерживают Unicode:

```swift
let latin: Character = "A"       // латиница
let cyrillic: Character = "Б"    // кириллица
let emoji: Character = "😀"       // смайлик
let arabic: Character = "ع"       // арабский
```

### Строка с разными символами:

```swift
let greeting = "Hello, Привет, 你好, 😀"
print(greeting)
```

---

## 3. Unicode Scalar

**Unicode scalar** — это базовый элемент Unicode.  
В Swift его тип — `Unicode.Scalar`.

```swift
let letter: Unicode.Scalar = "A"
print(letter.value) // 65 — код символа в Unicode
```

- Для кириллицы: `"Б".unicodeScalars.first!.value` → 1041
    
- Для эмодзи: `"😀".unicodeScalars.first!.value` → 128512
    

---

## 4. Работа со строками и Unicode

### Доступ к Unicode scalar

```swift
let text = "Привет"
for scalar in text.unicodeScalars {
    print(scalar.value)  // код каждого символа
}
```

### Символы и их подсчёт

```swift
let flag = "🇷🇺" // флаг России состоит из двух символов
print(flag.count)           // 1 character
print(flag.unicodeScalars.count) // 2 scalars
```

> Важно: один символ (`Character`) может состоять из нескольких Unicode scalar.

---

## 5. Преобразование к коду и обратно

```swift
// Код символа → Character
let scalar = Unicode.Scalar(0x1F600)! // 😀
let char = Character(scalar)
print(char)

// Character → Unicode
let code = "😀".unicodeScalars.first!.value
print(code) // 128512
```

---

## 6. Особенности Swift и Unicode

- Строки Swift — **Unicode корректные** (поддерживают emoji, диакритические знаки).
    
- Символы с разными кодами могут визуально совпадать (например, é = e + accent).
    
- Методы `.count` для строки считаются **Character**, а не Unicode scalar.
    

---

## 7. Итог

- **Unicode** = стандарт для кодирования символов.
    
- Swift `String` полностью его поддерживает.
    
- Можно работать на уровне:
    
    - `Character` → отдельный видимый символ
        
    - `Unicode.Scalar` → базовый код символа
        
- Поддержка emoji и разных языков встроена "из коробки".
    

---
