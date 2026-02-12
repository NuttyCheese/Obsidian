#swift #standard_library #higher_order_function #functional_programming #optional #collection #map #nil_filtering
`compactMap` — один из самых полезных и часто используемых методов в стандартной библиотеке [[Swift]]. Он одновременно **преобразует** элементы коллекции и **фильтрует** [[nil]]-значения.

Метод появился в **Swift 4.1** (2018 год) и практически сразу стал предпочтительным способом замены старого паттерна с [[map]] + [[filter]] или старого [[flatMap]], когда требовалось получить массив без `nil`.

## Что делает compactMap?

`compactMap`:
1. Применяет к каждому элементу коллекции переданное замыкание
2. Замыкание возвращает **опциональное** значение (`ElementOfResult?`)
3. Все `nil`-результаты автоматически отбрасываются
4. Возвращает массив **неопциональных** значений того типа, который вернуло замыкание

## Основная сигнатура

```swift
func compactMap<ElementOfResult>(
    _ transform: (Element) throws -> ElementOfResult?
) rethrows -> [ElementOfResult]
```

Существуют также варианты для других коллекций:

- `compactMap` на [[Sequence]] → возвращает `[ElementOfResult]`
- `compactMap` на [[Collection]] → возвращает `[ElementOfResult]`
- `compactMap` на [[Dictionary]] → возвращает `[ElementOfResult]`
- `compactMap` на [[Set]] → возвращает `[ElementOfResult]`

## Примеры использования

### 1. Преобразование строк в числа (классический пример)

```swift
let strings = ["12", "45", "seven", "19", "0", "abc", "88"]

let numbers = strings.compactMap { Int($0) }
// Результат: [12, 45, 19, 0, 88]
```

### 2. Работа с опциональными свойствами объектов

```swift
struct User {
    let name: String
    let ageString: String?
    let scoreString: String?
}

let users = [
    User(name: "Anna", ageString: "28", scoreString: "95"),
    User(name: "Ben",  ageString: nil,   scoreString: "82"),
    User(name: "Clara", ageString: "31", scoreString: nil),
    User(name: "David", ageString: "19", scoreString: "100")
]

let ages = users.compactMap { $0.ageString.flatMap(Int.init) }
// [28, 31, 19]

let scores = users.compactMap { user -> Int? in
    guard let str = user.scoreString else { return nil }
    return Int(str)
}
// [95, 82, 100]
```

### 3. Извлечение значений из опциональных массивов

```swift
let optionalArrays: [Int?] = [1, nil, 3, nil, 5, 6, nil, 8]

let numbers = optionalArrays.compactMap { $0 }
// [1, 3, 5, 6, 8]
```

### 4. Преобразование [[enum]] с [[associated value]]s

```swift
enum NetworkResponse {
    case success(data: String)
    case error(code: Int)
    case loading
}

let responses: [NetworkResponse] = [
    .success(data: "User data"),
    .error(code: 404),
    .loading,
    .success(data: "Profile"),
    .error(code: 500)
]

let successData = responses.compactMap { response -> String? in
    if case .success(let data) = response {
        return data
    }
    return nil
}
// ["User data", "Profile"]

let errorCodes = responses.compactMap { response -> Int? in
    if case .error(let code) = response {
        return code
    }
    return nil
}
// [404, 500]
```

### 5. compactMap + [[map]] в цепочке (очень частый паттерн)

```swift
struct Product {
    let name: String
    let priceString: String?
}

let products = [
    Product(name: "Phone",   priceString: "599"),
    Product(name: "Laptop",  priceString: nil),
    Product(name: "Headphones", priceString: "149"),
    Product(name: "Charger", priceString: "29")
]

let formattedPrices = products
    .compactMap { product -> Double? in
        product.priceString.flatMap(Double.init)
    }
    .map { price in
        String(format: "$%.2f", price)
    }
// ["$599.00", "$149.00", "$29.00"]
```

### 6. compactMap на [[Dictionary]]

```swift
let dict: [String: String?] = [
    "one":   "1",
    "two":   nil,
    "three": "3",
    "four":  "four",
    "five":  "5"
]

let numbers = dict.compactMap { key, value -> Int? in
    value.flatMap(Int.init)
}
// [1, 3, 5]
```

### 7. compactMap vs [[map]] + [[filter]] (для сравнения)

```swift
let values = ["10", "twenty", "30", "forty", "50"]

// Старый способ (до Swift 4.1 или когда нужен map + filter)
let oldWay = values
    .map { Int($0) }
    .filter { $0 != nil }
    .map { $0! }           // forced unwrap — не очень безопасно

// Современный и безопасный способ
let newWay = values.compactMap { Int($0) }
// [10, 30, 50]
```

## Когда использовать compactMap, а когда map + filter

| Задача                                                    | Рекомендуемый способ                                        | Почему                                          |
| --------------------------------------------------------- | ----------------------------------------------------------- | ----------------------------------------------- |
| Преобразование → отбрасывание [[nil]]                     | `compactMap`                                                | Коротко, безопасно, выразительно, одна операция |
| Преобразование → оставить nil                             | `map`                                                       | Нам нужны именно опциональные значения          |
| Фильтрация без преобразования                             | `filter`                                                    | `compactMap` здесь не нужен                     |
| Нужно и преобразовать и отфильтровать по сложному условию | `filter` + `map` или `compactMap` + дополнительный `filter` | зависит от читаемости кода                      |
| Работа с коллекцией опционалов                            | `compactMap { $0 }`                                         | Самый короткий и понятный способ убрать `nil`   |

## Важные замечания

- `compactMap` **не** является заменой `flatMap` в смысле «расплющивания» вложенных коллекций
- После Swift 4.1 старый `flatMap` (который делал `map` + `flat`) был **переименован** в `flatMap` → `joined()` / `flatMap` (для последовательностей)
- `compactMap` **всегда** возвращает **массив** (Array), даже если исходная коллекция была `Set` или `Dictionary`

## Полезный паттерн: compactMap + [[first]] / last / min / max

```swift
let dates = ["2024-01-15", "invalid", "2025-03-20", "2023-12-01", "bad"]

let sortedValidDates = dates
    .compactMap { ISO8601DateFormatter().date(from: $0) }
    .sorted()

if let earliest = sortedValidDates.first,
   let latest = sortedValidDates.last {
    print("Самая ранняя дата:", earliest)
    print("Самая поздняя дата:", latest)
}
```

`compactMap` — один из тех методов, которые делают код [[Swift]] более выразительным, безопасным и лаконичным. Его стоит использовать всегда, когда нужно преобразовать элементы коллекции и одновременно избавиться от `nil`-значений.
