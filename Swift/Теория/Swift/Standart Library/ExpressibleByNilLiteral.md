## 1. Что такое `ExpressibleByNilLiteral`

`ExpressibleByNilLiteral` — это **протокол стандартной библиотеки [[Swift]]**.

```swift
public protocol ExpressibleByNilLiteral {
    init(nilLiteral: ())
}
```

👉 **Смысл простой**:  
тип, который его реализует, **может быть инициализирован литералом [[nil]]**.

То есть компилятор _разрешает_ писать:

```swift
let value: SomeType = nil
```

если `SomeType` соответствует `ExpressibleByNilLiteral`.

---

## 2. Кто уже его реализует

Самый важный и очевидный пример:

```swift
Optional
```

```swift
extension Optional: ExpressibleByNilLiteral {
    public init(nilLiteral: ()) {
        self = .none
    }
}
```

Поэтому это работает:

```swift
let a: Int? = nil
let b: String? = nil
```

❗️Важно: **`nil` — это не `Optional.none`**, это _литерал_, который компилятор **превращает** в `.none`.

---

## 3. Что вообще такое `nil` в Swift

И вот тут ключевая мысль:

> **`nil` — это не значение, а литерал языка**

Как:

- `0`
    
- `"hello"`
    
- `true`
    

Но!

- `0` → `ExpressibleByIntegerLiteral`
    
- `"hello"` → `ExpressibleByStringLiteral`
    
- `nil` → `ExpressibleByNilLiteral`
    

### Компилятор думает так:

```swift
let x: Int? = nil
```

⬇️

1. Видит `nil`
    
2. Смотрит тип слева (`Int?`)
    
3. Проверяет: «умеет ли этот тип быть создан из `nil`?»
    
4. Да → вызывает `init(nilLiteral:)`
    
5. Получает `.none`
    

---

## 4. Можно ли реализовать самому?

Да. И это полезно **редко**, но метко.

### Пример: nullable wrapper

```swift
struct MyOptional<T>: ExpressibleByNilLiteral {
    let value: T?

    init(_ value: T?) {
        self.value = value
    }

    init(nilLiteral: ()) {
        self.value = nil
    }
}
```

Использование:

```swift
let a: MyOptional<Int> = nil
let b = MyOptional(10)
```

⚠️ Без `ExpressibleByNilLiteral` это **не скомпилируется**.

---

## 5. Зачем это вообще нужно

### 1️⃣ Синтаксический сахар

Позволяет писать API, которые **выглядят как Optional**, но ведут себя иначе.

### 2️⃣ DSL и инфраструктура

Часто используют в:

- Result Builders
    
- SwiftUI-подобных DSL
    
- Network / Query API
    

### 3️⃣ Совместимость с nil

Иногда нужно:

- принимать `nil`
    
- но **не быть Optional**
    

---

## 6. Почему `nil` нельзя присвоить обычному типу

```swift
let x: Int = nil // ❌
```

Потому что:

- `Int` **не реализует** `ExpressibleByNilLiteral`
    
- и **не имеет смысла** как концепция
    

Swift тут строг и последователен — как и старые добрые типизированные языки.

---

## 7. Как компилятор обрабатывает `nil` (упрощённо)

### Код:

```swift
let x: String? = nil
```

### Что реально происходит:

```swift
let x = Optional<String>(nilLiteral: ())
```

или логически:

```swift
let x = Optional<String>.none
```

### Важно

Компилятор **не хранит `nil` как значение**  
Он **заменяет его на `.none` на этапе компиляции**

---

## 8. `ExpressibleByNilLiteral` vs `Optional`

|Критерий|ExpressibleByNilLiteral|Optional|
|---|---|---|
|Что это|Протокол|Enum|
|Назначение|Разрешить `= nil`|Хранить `some / none`|
|Может существовать без Optional|✅|❌|
|Хранит значение|❌|✅|
|Имеет unwrap (`if let`)|❌|✅|
|Управляет памятью|❌|✅|

### Ключевая мысль

> `Optional` — это **тип данных**  
> `ExpressibleByNilLiteral` — это **разрешение компилятору**

---

## 9. Частая ошибка в понимании

❌ **Неправильно**:

> `nil` — это Optional.none

✅ **Правильно**:

> `nil` — это литерал, который _превращается_ в `.none`, если тип это позволяет

---

## 10. Почему Swift сделал именно так

Тут очень «традиционное» и правильное решение:

- `nil` не живёт в рантайме
    
- всё решается **на этапе компиляции**
    
- строгая типизация
    
- ноль магии в рантайме
    
- максимум предсказуемости в будущем
    

Это как с `ExpressibleByIntegerLiteral`:  
компилятор **не хранит `5`**, он вызывает `init(integerLiteral:)`.

---

## 11. Когда использовать самому

✅ Используй, если:

- делаешь DSL
    
- пишешь обёртки над Optional
    
- строишь декларативный API
    

❌ Не используй:

- «просто так»
    
- для бизнес-моделей
    
- как замену Optional
    

Optional — это фундамент языка. Его **не надо переизобретать**.

---

## 12. Короткий вывод (чеканно)

- `nil` — литерал
    
- `ExpressibleByNilLiteral` — разрешение
    
- `Optional` — реализация
    
- компилятор всё решает **до выполнения**
    
- Swift остаётся строгим и безопасным
    