**`if case`** — это мощная и очень часто используемая конструкция в [[Swift]] для **pattern matching** (соответствия шаблону) в условном операторе `if`.  
Она позволяет одновременно:

- проверить, соответствует ли значение определённому шаблону (case)  
- извлечь **[[associated value]]s** (ассоциированные значения)  
- сразу использовать их в блоке кода  

Это особенно удобно с [[enum]], [[Optional]] и любыми типами, поддерживающими pattern matching.

> Проще говоря:  
> `if case` = «если значение подходит под этот шаблон — извлеки данные и работай с ними, иначе пропусти или выйди».

### 1. Почему `if case` так любят в 2025–2026

| Ситуация / Проблема                         | Классический `switch` или `if let`   | `if case` (современный стиль)           | Почему `if case` выигрывает |
| ------------------------------------------- | ------------------------------------ | --------------------------------------- | --------------------------- |
| Хочу проверить только один case из [[enum]] | Нужно писать полный `switch`         | Одна строка `if case`                   | Краткость + читаемость      |
| Извлечь associated value из Optional        | `if let value = optional`            | `if case let value? = optional`         | Более выразительно          |
| Проверка + условие на значение              | `if let x = optional, x > 10`        | `if case let x? = optional, x > 10`     | Логика в одной строке       |
| Хочу избежать глубокой вложенности          | Много `if let` подряд → пирамида     | `if case` + `guard` или цепочка условий | Код остаётся плоским        |
| Работа с enum в цепочках или замыканиях     | Требует `switch` или отдельный метод | `if case` внутри замыкания — компактно  | Функциональный стиль        |

### 2. Полный синтаксис и все популярные варианты

#### Вариант 1 — Базовый `if case` с enum

```swift
enum NetworkResponse {
    case success(statusCode: Int, data: Data)
    case loading(progress: Double)
    case failure(Error)
}

let response = NetworkResponse.success(statusCode: 200, data: Data())

if case .success(let code, let data) = response {
    print("Успех! Код: \(code), размер данных: \(data.count) байт")
} else {
    print("Не успех")
}
```

#### Вариант 2 — `if case let` с Optional (альтернатива `if let`)

```swift
let optionalUser: User? = fetchUser()

if case let user? = optionalUser {
    print("Пользователь найден: \(user.name)")
} else {
    print("Пользователь не найден")
}

// Эквивалентно:
if let user = optionalUser {
    print("Пользователь найден: \(user.name)")
}
```

#### Вариант 3 — `if case` + дополнительные условия через `,`

```swift
let event: UIEvent? = getLatestEvent()

if case .click(let x, let y) = event, x > 100, y < 200 {
    print("Клик в правой верхней области: \(x),\(y)")
}
```

#### Вариант 4 — `if case` внутри замыкания / цепочки (очень популярно)

```swift
results.forEach { result in
    if case .success(let data) = result {
        process(data)
    }
}

// Или в map/filter
let successfulData = results.compactMap { result -> Data? in
    if case .success(let data) = result {
        return data
    }
    return nil
}
```

#### Вариант 5 — `if case` с диапазонами и паттернами

```swift
let score = 85

if case 80...100 = score {
    print("Отлично!")
}

let temperature = 38.5

if case 38... = temperature {
    print("Повышенная температура")
}
```

### 3. `if case` vs [[switch]] vs [[if let]] — когда что выбрать

| Задача / Сценарий                                    | Лучший выбор в 2026              | Почему                          |
| ---------------------------------------------------- | -------------------------------- | ------------------------------- |
| Проверяю только один case из enum                    | `if case`                        | Кратко, без полного switch      |
| Нужно обработать несколько case                      | `switch`                         | Полная проверка, exhaustiveness |
| Просто разворачиваю Optional без условий             | `if let` или `guard let`         | Классика, привычно              |
| Разворачиваю Optional + сразу условие                | `if case let value?, value > 10` | Более выразительно              |
| В цепочке методов ([[map]], [[filter]], [[forEach]]) | `if case` внутри замыкания       | Компактно и функционально       |
| Много вложенных проверок Optional                    | `guard let` + `if case`          | Плоский код                     |
| Хочу ранний выход при nil                            | `guard let` или `guard case let` | Early exit — лучший стиль       |

### 4. Лучшие практики `if case` в Swift 2026

- **Используй `if case`** вместо `switch`, когда тебе нужен **только один case**  
- **Комбинируй** с `,` для дополнительных условий — это очень читаемо  
- **В замыканиях** (`map`, `filter`, `forEach`) — `if case` часто короче и яснее  
- **Для Optional** — `if case let value? = optional` — современная альтернатива `if let`  
- **Не злоупотребляй** — если case-ов больше 2–3, лучше `switch`  
- **Swift 6 strict concurrency** — `if case` полностью безопасен  
- **Документируйте** — пиши комментарий «if case .success — обработка успешного ответа»

**Короткий девиз 2026**:
> `if case` — это «если значение подходит под этот шаблон — вот что с ним делать».  
> В 2026 году используй его:  
> - когда нужен только один case из enum  
> - для компактного разворачивания Optional с условиями  
> - внутри замыканий и цепочек методов  
> Это **самый выразительный** способ pattern matching в простых случаях.
