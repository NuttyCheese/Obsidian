## 1. Что такое `ExpressibleByIntegerLiteral`

Протокол стандартной библиотеки:

```swift
public protocol ExpressibleByIntegerLiteral {
    associatedtype IntegerLiteralType : _ExpressibleByBuiltinIntegerLiteral
    init(integerLiteral value: IntegerLiteralType)
}
```

Уже по сигнатуре видно: **всё серьёзнее**, чем с [[Bool]] и [[nil]].

👉 Тип, который его реализует, **может быть инициализирован целочисленным литералом**:

```swift
let x: SomeType = 42
```

---

## 2. Целочисленный литерал — это не `Int`

Ключевая мысль, которую часто упускают:

> `42` **не является `Int`**

`42` — это **integer literal**, сырой литерал без типа.

Тип определяется **контекстом**:

```swift
let a = 42        // Int
let b: Int8 = 42
let c: UInt = 42
```

Компилятор подбирает подходящий тип.

---

## 3. Почему тут есть [[associatedtype]]

Потому что:

- целые бывают **знаковые / беззнаковые**
    
- разной битности
    
- иногда **произвольной точности**
    

Поэтому Swift использует **внутренний тип**:

```swift
IntegerLiteralType
```

Обычно это что-то вроде `_MaxBuiltinIntegerType`.

Ты **почти никогда не работаешь с ним напрямую**.

---

## 4. Кто реализует `ExpressibleByIntegerLiteral`

Практически вся числовая иерархия:

- `Int`
    
- `Int8`, `Int16`, `Int32`, `Int64`
    
- `UInt`, `UInt8`, ...
    
- `Float`, `Double` (через конверсию)
    
- `CGFloat`
    
- `NSNumber`
    

И да — **не только целые типы**.

---

## 5. Как компилятор обрабатывает `42`

Код:

```swift
let x: Int = 42
```

Логически превращается в:

```swift
let x = Int(integerLiteral: 42)
```

Но реально:

- `42` сначала существует как **абстрактный literal**
    
- потом проверяется диапазон
    
- потом вызывается инициализатор
    

Если не влезает:

```swift
let x: Int8 = 1000 // ❌ compile-time error
```

💡 **Без рантайм-крашей. Старой школы строгость.**

---

## 6. Реализация своего типа (простой пример)

```swift
struct Counter: ExpressibleByIntegerLiteral {
    let value: Int

    init(integerLiteral value: Int) {
        self.value = value
    }
}
```

Использование:

```swift
let counter: Counter = 10
```

Компилируется и читается идеально.

---

## 7. Более правильная реализация (через Int)

⚠️ Важно: ты **не обязан** использовать `IntegerLiteralType` напрямую.

```swift
struct Level: ExpressibleByIntegerLiteral {
    let rawValue: Int

    init(integerLiteral value: Int) {
        self.rawValue = value
    }
}
```

Swift сам приведёт literal к [[Int]], если это возможно.

---

## 8. Пример с бизнес-смыслом

```swift
struct RetryCount: ExpressibleByIntegerLiteral {
    let maxAttempts: Int

    init(integerLiteral value: Int) {
        precondition(value >= 0)
        self.maxAttempts = value
    }
}
```

Использование:

```swift
let retries: RetryCount = 3
```

Читается:

> «3 попытки»

---

## 9. Опасное место ⚠️

```swift
let x = 1 + 2
```

👉 Это **не литерал**.

`ExpressibleByIntegerLiteral` применяется **только к литералам**, а не к результатам выражений.

---

## 10. Почему `BinaryInteger` — это другое

Частая путаница:

|ExpressibleByIntegerLiteral|BinaryInteger|
|---|---|
|Про инициализацию|Про арифметику|
|Работает на этапе компиляции|Работает в рантайме|
|Литералы|Операции|
|DSL|Математика|

Ты можешь:

```swift
struct MyInt: ExpressibleByIntegerLiteral {}
```

Но без `BinaryInteger` ты **не сможешь писать `+`, `-`, `*`**.

---

## 11. Почему `Double` принимает `42`

```swift
let x: Double = 42
```

Потому что:

- `42` — integer literal
    
- [[Double]] реализует `ExpressibleByIntegerLiteral`
    
- компилятор разрешает конверсию
    

Но:

```swift
let y: Double = 3.14 // уже другой протокол
```

---

## 12. Частая ошибка

❌ Делать тип «псевдо-интом»

```swift
struct BadInt: ExpressibleByIntegerLiteral {}
```

Без:

- диапазонов
    
- операций
    
- переполнения
    
- signed / unsigned
    

Такой тип быстро становится миной.

---

## 13. Когда использовать самому

✅ Используй, если:

- делаешь DSL
    
- хочешь читаемые конфиги
    
- выражаешь **смысл числа**
    

❌ Не используй:

- как замену `Int`
    
- в математике
    
- в низкоуровневом коде
    

---

## 14. Связь с прошлым и будущим

Исторически:

- `ExpressibleByIntegerLiteral` — мост от **безтиповых литералов** к строгой типизации
    

В будущем:

- Swift всё больше опирается на literal-протоколы в DSL
    
- но **числовая иерархия остаётся консервативной**
    

Это хороший баланс: новое — аккуратно, старое — надёжно.

---

## 15. Короткий итог

- `42` — не `Int`
    
- литералы типизируются контекстом
    
- `ExpressibleByIntegerLiteral` — входная точка
    
- `BinaryInteger` — поведение
    
- компилятор ловит ошибки **на этапе компиляции**
    

---
