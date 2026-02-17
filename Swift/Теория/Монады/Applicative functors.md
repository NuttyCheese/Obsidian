| Концепция               | Оператор    | Что позволяет делать                                                       | Пример в [[Swift]] (стандартный)     |
| ----------------------- | ----------- | -------------------------------------------------------------------------- | ------------------------------------ |
| **Functor**             | [[map]]     | Применить функцию к значению внутри контейнера                             | `Optional.map`, `Array.map`          |
| **Applicative Functor** | `<*>`       | Применить **функцию внутри контейнера** к значению в **другом контейнере** | `Optional.ap`, `Result.apply`        |
| **Monad**               | [[flatMap]] | Применить функцию, возвращающую контейнер                                  | `Optional.flatMap`, `Result.flatMap` |

**Коротко и по-человечески**:

- Functor: «У меня есть значение в коробке → достань и примени к нему функцию»  
- Applicative: «У меня есть функция в коробке и значение в другой коробке → достань функцию и примени её к значению»  
- Monad: «У меня есть значение в коробке → достань и примени функцию, которая сама вернёт новую коробку»

### 2. Классический пример: почему нужен Applicative

Представьте, что у вас есть две независимые `Optional`-величины и функция, которая их комбинирует:

```swift
let a: Int? = 5
let b: Int? = 3

// Хотим сложить a + b, но оба Optional
```

**Без Applicative** (громоздко):

```swift
let result: Int? = if let x = a, let y = b {
    x + y
} else {
    nil
}
```

**С Applicative** (красиво и выразительно):

```swift
let sum = { x in { y in x + y } } <*> a <*> b
// или короче
let sum = (+) <*> a <*> b
print(sum) // Optional(8)
```

### 3. Как реализовать `<*>` для стандартных типов

#### Для [[Optional]]

```swift
infix operator <*> { associativity left precedence 150 }

extension Optional {
    static func <*> <A, B>(f: (A) -> B?, a: Optional<A>) -> Optional<B> {
        switch (f, a) {
        case (.some(let fn), .some(let value)): return .some(fn(value))
        default: return .none
        }
    }
}

// Примеры использования
let plus: ((Int) -> Int)? = { $0 + 10 }
let num: Int? = 5

print(plus <*> num)          // Optional(15)

let add: (Int) -> (Int) -> Int = { x in { y in x + y } }
let a: Int? = 7
let b: Int? = 8

print(add <*> a <*> b)       // Optional(15)
print((+) <*> a <*> b)       // Optional(15)
```

#### Для Result (очень популярно в 2026)

```swift
infix operator <*> { associativity left precedence 150 }

extension Result {
    static func <*> <A, B>(f: Result<(A) -> B, Failure>, a: Result<A, Failure>) -> Result<B, Failure> {
        switch (f, a) {
        case (.success(let fn), .success(let value)): return .success(fn(value))
        case (.failure(let error), _): return .failure(error)
        case (_, .failure(let error)): return .failure(error)
        }
    }
}

// Примеры
let successFn: Result<(Int) -> Int, Error> = .success { $0 * 2 }
let successVal: Result<Int, Error> = .success(10)
print(successFn <*> successVal)  // .success(20)

let failure: Result<Int, Error> = .failure(NSError(domain: "", code: -1))
print(successFn <*> failure)     // .failure
```

### 4. Реальные сценарии в iOS-разработке 2026

#### Сценарий 1 — Комбинирование нескольких асинхронных операций ([[Combine]] / [[async]])

```swift
let userIDPublisher = fetchUserID()     // AnyPublisher<Int, Error>
let tokenPublisher  = fetchToken()      // AnyPublisher<String, Error>

let credentials = { id in { token in (id, token) } }
    <*> userIDPublisher
    <*> tokenPublisher

// Теперь credentials — это издатель пары (userID, token)
```

#### Сценарий 2 — Валидация формы (несколько полей)

```swift
struct FormData {
    let email: String?
    let password: String?
    let age: Int?
}

let email: String? = "user@example.com"
let password: String? = "pass123"
let age: Int? = 25

let form = { email in { password in { age in FormData(email: email, password: password, age: age) } } }
    <*> email
    <*> password
    <*> age
```

#### Сценарий 3 — Комбинирование Result в бизнес-логике

```swift
let userResult: Result<User, APIError> = fetchUser()
let configResult: Result<Config, APIError> = fetchConfig()

let combined = { user in { config in (user, config) } }
    <*> userResult
    <*> configResult

switch combined {
case .success(let (user, config)):
    // оба успешны
case .failure(let error):
    // хотя бы один провалился
}
```

### 5. Сравнение Functor vs Applicative vs Monad

| Концепция          | Оператор | Что позволяет                              | Пример в Swift |
|--------------------|----------|--------------------------------------------|----------------|
| Functor            | `map`    | f(a) → F<b>                                | `Optional.map`, `Result.map` |
| Applicative        | `<*>`    | F<(a → b)> <*> F<a> → F<b>                 | `<*>` для Optional, Result |
| Monad              | `flatMap`| F<a> → (a → F<b>) → F<b>                   | `Optional.flatMap`, `Result.flatMap` |

**Коротко**:
- Functor: одна функция + один контейнер  
- Applicative: функция в контейнере + несколько контейнеров  
- Monad: функция возвращает контейнер (flatMap)

### 6. Лучшие практики 2026

- Используйте `<*>` для комбинирования **независимых** контейнеров  
- Для **зависимых** операций (один результат влияет на следующий) — используйте `flatMap` / `async let` / `TaskGroup`  
- В Combine — `<*>` эквивалентен `combineLatest` + `map`  
- В async/await — используйте `async let` или `TaskGroup` вместо `<*>`  
- Реализуйте `<*>` для своих типов (Result, Future, IO и т.д.) — это делает код выразительным  
- Для коллекций — рассмотрите `zip` вместо `<*>` (более семантично)

**Короткий девиз 2026**:
> «Applicative Functor — это когда ты хочешь сложить, умножить или объединить несколько контейнеров одновременно, не вынимая значения вручную.  
> Если операции независимы — `<*>` твоя лучшая подруга.  
> Если одна зависит от другой — бери `flatMap` / `async let` / Monad.»
