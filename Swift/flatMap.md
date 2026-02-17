`flatMap` — один из самых мощных, но при этом один из самых **часто неправильно понимаемых** методов в стандартной библиотеке [[Swift]].

С 2018–2019 годов (Swift 4.1 → Swift 5) поведение и рекомендуемое использование `flatMap` сильно изменилось. Сегодня в современном Swift (2025–2026) существуют **три основных сценария использования** `flatMap`, и очень важно различать их.

## Три разных смысла flatMap в современном Swift

| Вариант                        | Когда появился | Что делает                                       | Возвращает        | Современная рекомендация (2026)          |
| ------------------------------ | -------------- | ------------------------------------------------ | ----------------- | ---------------------------------------- |
| `flatMap` для опционалов → [T] | Swift 1.x      | [[map]] + compact (удаляет [[nil]])              | [ElementOfResult] | **Устарел**. Используйте [[compactMap]]  |
| `flatMap` для коллекций → [T]  | Swift 1.x      | map + flatten (расплющивает вложенные коллекции) | [ElementOfResult] | **Актуален**. Это основной смысл сегодня |
| `flatMap` на Optional<[T]>     | Swift 1.x      | Расплющивает опциональный массив                 | [T]?              | **Актуален**, но редко нужен             |

**Самое важное правило 2025–2026 годов**:

- Хотите преобразовать и **убрать nil** → **всегда** используйте `compactMap`
- Хотите преобразовать и **расплющить** вложенные коллекции → используйте `flatMap`
- Хотите просто убрать nil без преобразования → `compactMap { $0 }`

## Актуальные сигнатуры (Swift 6+)

```swift
// 1. Самый частый сегодня — расплющивание коллекций
func flatMap<ElementOfResult>(
    _ transform: (Element) throws -> ElementOfResult
) rethrows -> [ElementOfResult]
    where ElementOfResult: Sequence

// 2. Устаревший вариант (для опционалов) — компакт-маппинг
@available(*, deprecated, renamed: "compactMap(_:)")
func flatMap<ElementOfResult>(
    _ transform: (Element) throws -> ElementOfResult?
) rethrows -> [ElementOfResult]
```

## Примеры — правильное и современное использование

### 1. Расплющивание вложенных массивов (самый актуальный сценарий)

```swift
let arrayOfArrays = [[1, 2], [3, 4, 5], [], [6]]

let flat = arrayOfArrays.flatMap { $0 }
// → [1, 2, 3, 4, 5, 6]

let wordsByLine = [
    ["Swift", "is"],
    ["very", "expressive"],
    ["and", "fast"]
]

let allWords = wordsByLine.flatMap { $0 }
// → ["Swift", "is", "very", "expressive", "and", "fast"]
```

### 2. Преобразование + расплющивание

```swift
struct Department {
    let name: String
    let employees: [String]
}

let company = [
    Department(name: "iOS", employees: ["Anna", "Ben"]),
    Department(name: "Android", employees: []),
    Department(name: "Backend", employees: ["Clara", "David", "Emma"])
]

let allEmployees = company.flatMap { $0.employees }
// → ["Anna", "Ben", "Clara", "David", "Emma"]
```

### 3. flatMap на [[Optional]]<[T]>

```swift
let optionalArray: [Int]? = [1, 2, 3]

let flattened = optionalArray.flatMap { $0 }
// Тип: [Int]?
// → [1, 2, 3]

let nilArray: [Int]? = nil
let empty = nilArray.flatMap { $0 }  // → nil
```

### 4. Старый (устаревший) паттерн — compact mapping

```swift
// ❌ Не рекомендуется в 2025–2026
let oldWay = ["1", "2", "three", "4"].flatMap { Int($0) }
// → [1, 2, 4]

// ✓ Современный и правильный способ
let correct = ["1", "2", "three", "4"].compactMap { Int($0) }
// → [1, 2, 4]
```

### 5. Реальные сценарии использования flatMap (2026)

```swift
// Получить все теги из массива постов
struct Post {
    let title: String
    let tags: [String]
}

let posts: [Post] = // ...

let allTags = posts.flatMap { $0.tags }

// Получить все координаты из маршрутов
struct Route {
    let points: [CLLocationCoordinate2D]
}

let routes: [Route] = // ...

let allCoordinates = routes.flatMap { $0.points }

// Все продукты из всех заказов
let allProducts = orders.flatMap { $0.items }
```

### 6. Сравнение производительности и стиля

```swift
// Вариант 1 — самый понятный и эффективный
let flat1 = nested.flatMap { $0 }

// Вариант 2 — то же самое, но более многословно
let flat2 = nested.reduce(into: [Int]()) { $0 += $1 }

// Вариант 3 — самый медленный и ненужный
let flat3 = nested.joined().map { $0 }   // joined() возвращает FlattenSequence
```

## Когда использовать что (рекомендации 2026)

| Задача                                           | Метод             | Почему                                                                 |
|--------------------------------------------------|-------------------|------------------------------------------------------------------------|
| Убрать `nil` из `[T?]`                           | `compactMap { $0 }` | Самый короткий и семантически точный способ                            |
| Преобразовать `[String]` → `[Int]` и убрать nil  | `compactMap { Int($0) }` | Явно говорит: «преобразую и отбрасываю неудачи»                        |
| Расплющить `[[T]]` → `[T]`                       | `flatMap { $0 }`  | Чётко выражает намерение «расплющить»                                  |
| Преобразовать и расплющить `[T]` → `[U]`         | `flatMap { ... }` | Когда transform возвращает коллекцию                                   |
| Преобразовать и **оставить nil**                 | `map`             | `flatMap` / `compactMap` здесь не нужны                                |

## Итог — современные правила использования flatMap

- **Основной смысл в 2026 году** — **расплющивание** вложенных последовательностей
- Если вы видите `flatMap`, за которым сразу идёт `Int($0)` / `URL($0)` / `Date($0)` → **99% случаев это ошибка** → замените на `compactMap`
- `flatMap { $0 }` — отличный идиоматичный способ расплющить массив массивов
- `joined()` — альтернатива, когда не нужно преобразование, но часто менее читаема

Правильное использование `flatMap` и `compactMap` — один из самых заметных признаков современного, идиоматичного Swift-кода в 2025–2026 годах.