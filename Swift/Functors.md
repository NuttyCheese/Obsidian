Функтор — это **контейнер** (или контекст), который умеет **применять функцию к своему содержимому**, не вынимая это содержимое наружу и не меняя саму оболочку.

Главный оператор функтора — **[[map]]**.

**Коротко и по-человечески**:

- У тебя есть **коробка** ([[Optional]], [[Array]], [[Result]], [[Publisher]] и т.д.)  
- Внутри коробки лежит **значение** (или ничего, или ошибка)  
- Ты говоришь: «Возьми функцию и примени её к тому, что внутри коробки»  
- Функтор сам решает: если внутри что-то есть → применяет функцию, если ничего → возвращает пустую коробку

### Основные функторы в стандартной библиотеке Swift

| Тип              | Оператор map | Что делает map                                                                 | Когда возвращает nil / failure |
|------------------|--------------|--------------------------------------------------------------------------------|--------------------------------|
| `Optional<T>`    | `map`        | Применяет функцию к значению, если оно есть                                    | Если был `.none`               |
| `Array<T>`       | `map`        | Применяет функцию к каждому элементу                                           | Никогда (всегда новый массив)  |
| `Result<Success, Failure>` | `map` | Применяет функцию только к `.success`                                          | Если был `.failure`            |
| `Publisher` (Combine) | `map` | Преобразует выходные значения потока                                           | Не влияет на ошибки (см. `mapError`) |
| `Task` / `async let` (async/await) | — | Нет встроенного map, но можно использовать `await` + трансформацию            | —                              |

### Полные примеры кода

#### 1. [[Optional]] — самый классический функтор

```swift
let number: Int? = 42
let doubled = number.map { $0 * 2 }           // Optional(84)
let stringified = number.map { String($0) }   // Optional("42")
let nilValue: Int? = nil
let nilMapped = nilValue.map { $0 * 100 }     // nil
```

#### 2. [[Array]] — самый часто используемый

```swift
let names = ["anna", "bob", "clara"]
let capitalized = names.map { $0.capitalized }          // ["Anna", "Bob", "Clara"]
let lengths = names.map(\.count)                        // [4, 3, 5] — key path
let doubled = names.map { $0 + $0 }                     // ["annaanna", "bobbob", "claraclara"]
```

#### 3. Result — самый полезный в бизнес-логике

```swift
enum APIError: Error { case invalidData }

let success: Result<String, APIError> = .success("42")
let mapped = success.map { Int($0) ?? 0 }               // .success(42)

let failure: Result<String, APIError> = .failure(.invalidData)
let failedMapped = failure.map { $0.count }             // .failure(.invalidData)
```

#### 4. Комбинирование нескольких функторов (Applicative Functor)

```swift
let a: Int? = 5
let b: Int? = 3

let add = { x in { y in x + y } }
let sum = add <*> a <*> b          // Optional(8)

// Или короче с оператором + (если определить)
let sumShort = (+) <*> a <*> b     // Optional(8)
```

#### 5. Функтор в [[Combine]] ([[Publisher]])

```swift
import Combine

let numbers = [1, 2, 3, 4, 5].publisher
let squared = numbers.map { $0 * $0 }  // 1, 4, 9, 16, 25
```

### 6. Почему функторы полезны (реальные преимущества 2026)

- **Безопасность** — нет принудительного unwrapping (`!`) и force try  
- **Чистота кода** — цепочки [[map]] / [[flatMap]] / [[compactMap]] вместо [[if let]] / [[guard let]]  
- **Композиция** — легко комбинировать преобразования  
- **Функциональный стиль** — меньше мутабельных переменных  
- **Производительность** — компилятор часто оптимизирует `map` до простого цикла  
- **Тестируемость** — легко мокать контейнеры (Optional, Result, Publisher)

### 7. Сравнение Functor vs Applicative vs Monad (2026)

| Концепция          | Оператор | Что позволяет                              | Пример в Swift |
|--------------------|----------|--------------------------------------------|----------------|
| Functor            | `map`    | f(a) → F<b>                                | `Optional.map`, `Result.map` |
| Applicative        | `<*>`    | F<(a → b)> <*> F<a> → F<b>                 | `<*>` для Optional, Result |
| Monad              | `flatMap`| F<a> → (a → F<b>) → F<b>                   | `Optional.flatMap`, `Result.flatMap` |

**Коротко**:
- Functor: одна функция + один контейнер  
- Applicative: функция в контейнере + несколько контейнеров  
- Monad: функция возвращает контейнер (flatMap)

### 8. Лучшие практики 2026

- Используйте `map` вместо `if let` / `guard let` там, где возможно  
- Для нескольких значений — используйте **Applicative** (`<*>`), а не nested `map`  
- Для зависимых операций — используйте **Monad** (`flatMap`)  
- В Combine — `map`, `tryMap`, `mapError`, `compactMap`, `filter` — ваши лучшие друзья  
- В async/await — используйте `map` через `await` + трансформацию  
- Для коллекций — `map`, `compactMap`, `flatMap` — стандарт  
- Избегайте force unwrap (`!`) — используйте `map` / `flatMap` / `??`  
- В SwiftUI — `map` часто используется для трансформации `@Published` / `@State`

**Короткий девиз 2026**:
> «Функтор — это когда ты говоришь контейнеру: «Возьми мою функцию и примени её к тому, что у тебя внутри».  
> Не доставай значение руками — пусть контейнер сам сделает это безопасно и красиво.»
