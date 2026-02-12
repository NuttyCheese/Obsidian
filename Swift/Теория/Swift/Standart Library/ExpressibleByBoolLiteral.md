## 1. Что такое `ExpressibleByBoolLiteral`

Это протокол стандартной библиотеки [[Swift]]:

```swift
public protocol ExpressibleByBoolLiteral {
    init(booleanLiteral value: Bool)
}
```

👉 Тип, который его реализует, **можно инициализировать литералом `true` или `false`**.

То есть компилятор разрешает:

```swift
let x: SomeType = true
let y: SomeType = false
```

если `SomeType: ExpressibleByBoolLiteral`.

---

## 2. Кто уже его реализует

### Очевидный кандидат — [[Bool]]

```swift
extension Bool: ExpressibleByBooleanLiteral {
    public init(booleanLiteral value: Bool) {
        self = value
    }
}
```

Но тут интереснее другое 👇  
**`Bool` — не единственный возможный смысл `true / false`.**

---

## 3. `true` / `false` — это тоже литералы

Так же, как:

- [[nil]]
    
- `5`
    
- `"text"`
    

`true` и `false` — **булевы литералы**, а не «магические значения».

Компилятор работает так:

```swift
let flag: Bool = true
```

⬇️

```swift
let flag = Bool(booleanLiteral: true)
```

---

## 4. Можно ли реализовать самому? Да, и вот тут магия

### Пример 1: Семантический флаг

```swift
struct FeatureToggle: ExpressibleByBoolLiteral {
    let isEnabled: Bool

    init(booleanLiteral value: Bool) {
        self.isEnabled = value
    }
}
```

Использование:

```swift
let feature: FeatureToggle = true
let legacyMode: FeatureToggle = false
```

Читается **как декларация смысла**, а не как данные.

---

## 5. Почему это полезно

### 1️⃣ Сильная семантика

Ты кодируешь **намерение**, а не тип.

```swift
func startTracking(_ enabled: FeatureToggle) { }
```

Вызов:

```swift
startTracking(true)
```

Читается:

> «включи трекинг»

---

### 2️⃣ DSL / конфигурации

Очень часто используется в:

- SwiftUI
    
- Result Builders
    
- declarative API
    

```swift
.padding(true)
```

Внутри — не `Bool`, а **контекстное решение**.

---

## 6. Пример посложнее: состояние, а не логика

```swift
enum Permission: ExpressibleByBoolLiteral {
    case allowed
    case denied

    init(booleanLiteral value: Bool) {
        self = value ? .allowed : .denied
    }
}
```

Использование:

```swift
let permission: Permission = true
```

🔥 Вот тут важно:

- `true` ≠ `Bool`
    
- `true` → **бизнес-смысл**
    

---

## 7. Как компилятор выбирает тип

Классическая ситуация:

```swift
let x = true
```

Тип?

👉 `Bool`

Но:

```swift
let x: FeatureToggle = true
```

Компилятор:

1. Видит `true`
    
2. Видит ожидаемый тип
    
3. Проверяет `ExpressibleByBoolLiteral`
    
4. Вызывает `init(booleanLiteral:)`
    

❗️**Контекст решает всё**

---

## 8. Можно ли перегрузить поведение `true`?

Нет.  
Ты **не меняешь `true`**, ты меняешь **как тип реагирует на литерал**.

Это фундаментальная разница.

---

## 9. Сравнение: `Bool` vs `ExpressibleByBoolLiteral`

|Критерий|Bool|ExpressibleByBoolLiteral|
|---|---|---|
|Тип данных|Да|Нет|
|Хранит значение|Да|Нет|
|Логические операции (`&&`)|Да|Нет|
|Можно писать `= true`|Да|Да|
|Контекстно-зависим|Нет|Да|

👉 Протокол — это **входные ворота**, а не логика.

---

## 10. Типичная ошибка

❌ Пытаться заменить `Bool`

```swift
func setFlag(_ value: ExpressibleByBoolLiteral) // ❌
```

Протокол **не про полиморфизм**, а про **инициализацию**.

---

## 11. Связь с другими literal-протоколами

| Литерал        | Протокол                        |
| -------------- | ------------------------------- |
| `true / false` | ExpressibleByBoolLiteral        |
| `nil`          | [[ExpressibleByNilLiteral]]     |
| `42`           | [[ExpressibleByIntegerLiteral]] |
| `"text"`       | [[ExpressibleByStringLiteral]]  |

Все они работают **одинаково**:

> компилятор → init(literal:)

---

## 12. Когда использовать в реальном коде

✅ Используй, если:

- делаешь декларативный API
    
- нужен читаемый DSL
    
- хочешь выразить смысл, а не тип
    

❌ Не используй:

- в бизнес-моделях
    
- вместо `Bool`
    
- если читаемость не улучшается
    

Как и раньше — **[[Optional]] трогать не надо, Bool тоже** 🙂

---

## 13. Короткий итог

- `true / false` — литералы
    
- `ExpressibleByBoolLiteral` — контракт с компилятором
    
- контекст решает, во что превратится литерал
    
- используется ради **смысла**, а не ради экономии кода
    
