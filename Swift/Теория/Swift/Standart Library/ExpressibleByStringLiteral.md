## 1. Что такое `ExpressibleByStringLiteral`

Протокол стандартной библиотеки [[Swift]]:

```swift
public protocol ExpressibleByStringLiteral {
    associatedtype StringLiteralType : _ExpressibleByBuiltinStringLiteral
    init(stringLiteral value: StringLiteralType)
}
```

👉 Тип, который его реализует, **можно инициализировать строковым литералом**:

```swift
let x: SomeType = "hello"
```

---

## 2. Строковый литерал — это не [[String]]

Ключевая мысль (аналогично 42 ≠ [[Int]]):

> `"hello"` **не является `String`**

Это **string literal**, сырой литерал без конкретного типа.

Тип появляется **из контекста**:

```swift
let a = "hi"           // String
let b: Substring = "hi"
let c: StaticString = "hi"
```

Один и тот же литерал → разные типы.

---

## 3. Почему [[associatedtype]] опять здесь

Строки в Swift — сложная штука:

- UTF-8 / UTF-16 / Unicode Scalars
    
- [[compile-time]] строки
    
- runtime строки
    
- null-terminated C-строки
    

Поэтому используется **встроенный тип компилятора**:

```swift
StringLiteralType
```

Ты с ним **никогда напрямую не работаешь**.

---

## 4. Кто реализует `ExpressibleByStringLiteral`

Основные типы:

- `String`
    
- `Substring`
    
- `StaticString`
    
- `Character`
    
- `NSString`
    
- `CFString`
    

И множество DSL-типов.

---

## 5. Как компилятор обрабатывает `"hello"`

Код:

```swift
let s: String = "hello"
```

Логически:

```swift
let s = String(stringLiteral: "hello")
```

На практике:

- литерал хранится в read-only сегменте
    
- может быть deduplicated
    
- может быть встроен напрямую в бинарник
    

💡 **Никакой магии в рантайме.**

---

## 6. Подпротоколы (важно!)

На самом деле их **три**:

```swift
ExpressibleByStringLiteral
ExpressibleByExtendedGraphemeClusterLiteral
ExpressibleByUnicodeScalarLiteral
```

### Зачем?

```swift
let c: Character = "A" // OK
let c: Character = "AB" // ❌
```

`Character` реализует **более строгий** протокол.

---

## 7. Реализация своего типа (базовый пример)

```swift
struct UserName: ExpressibleByStringLiteral {
    let value: String

    init(stringLiteral value: String) {
        self.value = value
    }
}
```

Использование:

```swift
let name: UserName = "alex"
```

Читается идеально.

---

## 8. Реализация с валидацией (🔥 важно)

```swift
struct Email: ExpressibleByStringLiteral {
    let value: String

    init(stringLiteral value: String) {
        precondition(value.contains("@"), "Invalid email")
        self.value = value
    }
}
```

⚠️ Это **compile-time syntax**, но **[[runtime]] проверка**.

Swift пока не делает полноценную compile-time валидацию строк.

---

## 9. Почему `StaticString` — особенный

```swift
let s: StaticString = "Hello"
```

- строка известна **на этапе компиляции**
    
- не аллоцируется
    
- часто используется в:
    
    - `assert`
        
    - `fatalError`
        
    - `os_log`
        

Но:

```swift
let s: StaticString = String("Hello") // ❌
```

Только литералы.

---

## 10. Опасное место ⚠️

```swift
let x = "a" + "b"
```

Это **не литерал**, а выражение.

`ExpressibleByStringLiteral` тут **не участвует**.

---

## 11. `ExpressibleByStringLiteral` vs `LosslessStringConvertible`

Частая путаница:

|ExpressibleByStringLiteral|LosslessStringConvertible|
|---|---|
|Работает с `"text"`|Работает с `String`|
|compile-time|runtime|
|DSL|парсинг|
|литералы|данные|

---

## 12. Типичная ошибка

❌ Использовать как замену `String`

```swift
struct BadString: ExpressibleByStringLiteral {}
```

Без:

- локализации
    
- сравнения
    
- нормализации
    
- производительности
    

Это быстро ломается.

---

## 13. Когда использовать самому

✅ Используй, если:

- делаешь DSL
    
- конфиги
    
- ключи, идентификаторы, теги
    
- compile-time выражение смысла
    

❌ Не используй:

- для пользовательского ввода
    
- для парсинга
    
- для динамических строк
    

---

## 14. Связь с прошлым и будущим

Прошлое:

- строки — всегда были болью
    
- Swift сделал их безопасными, но сложными
    

Будущее:

- макросы
    
- compile-time evaluation
    
- возможно, настоящая compile-time валидация
    

`ExpressibleByStringLiteral` — **мост** к этому будущему.

---

## 15. Короткий итог

- `"hello"` — не `String`
    
- это литерал
    
- тип решается контекстом
    
- протокол — разрешение компилятору
    
- идеален для DSL и семантики
    

---

Если продолжать логично:

- `ExpressibleByArrayLiteral`
    
- `ExpressibleByDictionaryLiteral`
    
- или разобрать **почему `String` — коллекция, но `Character` — нет**
    
- или связать всё в одну картину literal-протоколов
    

Куда идём дальше?