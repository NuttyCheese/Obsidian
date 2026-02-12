## 1. Что такое `String`

**`String`** — это **тип данных в [[Swift]] для хранения текста**.

- Представляет собой **последовательность символов** ([[Character]])
    
- Поддерживает **[[Unicode]]**, включая эмодзи, специальные символы и разные языки
    
- [[Value Type]] — **копируется при присваивании** ([[struct]])
    

> Проще говоря: String = «текстовая последовательность символов».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**Character**|Один символ строки|
|**Unicode**|Стандарт кодирования символов, который поддерживает Swift|
|**Interpolation**|Вставка переменных в строку через `\(variable)`|
|**Concatenation**|Склеивание строк через `+`|
|**Substring**|Часть строки, получаемая срезом|
|**Index**|Позиция символа в строке (String.Index)|
|**Count**|Количество символов в строке|
|**Multiline string**|Строка, занимающая несколько строк (`"""`)|

---

## 3. Основной синтаксис

```swift
let greeting: String = "Hello, World!"
let name = "Alice"
let message = "Hello, \(name)!"
```

- Тип можно указать явно или использовать **type inference**
    
- Поддерживается **интерполяция строк**
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простейшая строка

```swift
let greeting = "Hello"
print(greeting) // Hello
```

- Immutable — [[let]] нельзя менять
    
- Для изменяемой строки используем [[var]]
    

---

### Пример 2. Конкатенация и интерполяция

```swift
let firstName = "John"
let lastName = "Doe"
let fullName = firstName + " " + lastName
let greeting = "Hello, \(fullName)!"
print(greeting) // Hello, John Doe!
```

- `+` для склеивания строк
    
- `\(variable)` для вставки значения в строку
    

---

### Пример 3. Многострочная строка

```swift
let poem = """
Roses are red,
Violets are blue,
Swift is fun,
And so are you.
"""
print(poem)
```

- Строки можно переносить на несколько строк
    
- Сохраняются переносы и форматирование
    

---

### Пример 4. Работа с символами и индексами

```swift
let text = "Hello"
for char in text {
    print(char)
}

let startIndex = text.startIndex
let firstChar = text[startIndex]
print(firstChar) // H
```

- Строка — последовательность `Character`
    
- Доступ к символу через `String.Index`
    

---

### Пример 5. Срезы (Substring)

```swift
let text = "Hello, World!"
let start = text.index(text.startIndex, offsetBy: 7)
let end = text.index(text.startIndex, offsetBy: 11)
let substring = text[start...end]
print(substring) // World
```

- Срез строки создаёт `Substring`
    
- Можно преобразовать обратно в `String`
    

---

### Пример 6. Методы String

```swift
let message = "  Hello Swift  "
print(message.uppercased())      // HELLO SWIFT
print(message.lowercased())      //   hello swift  
print(message.trimmingCharacters(in: .whitespaces)) // Hello Swift
print(message.contains("Swift")) // true
```

- Поддерживаются **uppercased, lowercased, trimming, contains** и многие другие методы
    

---

### Пример 7. Unicode и Emoji

```swift
let emoji = "👩‍💻"
print(emoji.count) // 1
for char in emoji {
    print(char)
}
```

- Swift корректно обрабатывает **Unicode символы и emoji**
    
- `count` учитывает символы, а не байты
    

---

## 5. Особенности String

1. **Value type (struct)** — копируется при присваивании
    
2. **Unicode корректность** — работает с разными языками и emoji
    
3. **Immutable по умолчанию** — для изменения нужно `var`
    
4. Поддерживает **интерполяцию, конкатенацию и методы**
    
5. Используется для **всего текстового контента, логов, UI, сетевых данных**
    

---

## 6. Итог

- **String** = тип для текста, последовательность символов
    
- Поддерживает **Unicode, интерполяцию, конкатенацию, срезы, методы**
    
- Основной инструмент работы с текстом в Swift
    

---
