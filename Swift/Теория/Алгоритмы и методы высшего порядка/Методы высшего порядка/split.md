#algoritms #Swift #higher-order_func 
# split(separator:maxSplits:omittingEmptySubsequences:) в [[Swift]]

Метод `split` — один из самых удобных и часто используемых способов разбить строку на подстроки по разделителю (или по условию).

Он возвращает **ленивую коллекцию** типа `Substring` (не `[String]`!), что делает его очень эффективным по памяти.

## Основные сигнатуры (Swift 5.7–6.x)

```swift
// Самый частый вариант
func split(
    separator: Character,
    maxSplits: Int = Int.max,
    omittingEmptySubsequences: Bool = true
) → some Collection<Substring>

// По любому разделителю (строка, символ, последовательность)
func split<S: StringProtocol>(
    separator: S,
    maxSplits: Int = Int.max,
    omittingEmptySubsequences: Bool = true
) → some Collection<Substring>

// Самый мощный — с замыканием (по условию)
func split(
    maxSplits: Int = Int.max,
    omittingEmptySubsequences: Bool = true,
    whereSeparator isSeparator: (Character) throws → Bool
) rethrows → some Collection<Substring>
```

## Ключевые особенности

- Возвращает **Substring**, а не [[String]] → очень экономит память
- **Ленивый** — элементы вычисляются только при обращении
- **Не мутирует** исходную строку
- По умолчанию **игнорирует пустые подстроки** (`omittingEmptySubsequences: true`)
- `maxSplits` ограничивает количество разбиений (очень полезно при парсинге CSV, логов и т.д.)

## Примеры использования

### 1. Классическое разбиение по символу

```swift
let csv = "name,age,city"
let fields = csv.split(separator: ",")
// → ["name", "age", "city"]  типа Collection<Substring>
```

### 2. Разбиение по строке-разделителю

```swift
let path = "/Users/alice/Documents/report.pdf"
let components = path.split(separator: "/")
// → ["", "Users", "alice", "Documents", "report.pdf"]
```

### 3. Учёт пустых полей (CSV с пропусками)

```swift
let row = "value1,,value3,,value5"
let withEmpties = row.split(separator: ",", omittingEmptySubsequences: false)
// → ["value1", "", "value3", "", "value5"]

let withoutEmpties = row.split(separator: ",")
// → ["value1", "value3", "value5"]
```

### 4. Ограничение количества разбиений (maxSplits)

```swift
let log = "ERROR:2025-02-06 14:35:12:Database connection failed: timeout"
let parts = log.split(separator: ":", maxSplits: 2)
// → ["ERROR", "2025-02-06 14:35:12", "Database connection failed: timeout"]
```

### 5. Разбиение по любому пробельному символу (очень частый паттерн)

```swift
let text = "Swift    is\tvery  expressive\nand\tfast"
let words = text.split { $0.isWhitespace }
// → ["Swift", "is", "very", "expressive", "and", "fast"]
```

### 6. Разбиение на строки фиксированной длины (chunking)

```swift
extension String {
    func chunks(of length: Int) → some Collection<Substring> {
        sequence(state: startIndex) { index in
            guard index < endIndex else { return nil }
            let end = index(distance: length, limitedBy: endIndex) ?? endIndex
            defer { index = end }
            return self[index..<end]
        }
    }
}

let hex = "a1b2c3d4e5f67890"
for chunk in hex.chunks(of: 4) {
    print(chunk)  // "a1b2", "c3d4", "e5f6", "7890"
}
```

### 7. Парсинг [[URL]]-параметров

```swift
let url = "https://example.com/search?q=swift+programming&lang=ru&page=3"
if let query = url.split(separator: "?").last {
    let params = query.split(separator: "&")
        .compactMap { param -> (String, String)? in
            let parts = param.split(separator: "=", maxSplits: 1)
            guard parts.count == 2 else { return nil }
            return (String(parts[0]), String(parts[1]))
        }
    
    print(params) // [("q", "swift+programming"), ("lang", "ru"), ("page", "3")]
}
```

### 8. Разбиение по нескольким разделителям

```swift
let messy = "one;two,three four;five"
let tokens = messy.split { "; ,".contains($0) }
// → ["one", "two", "three", "four", "five"]
```

## Важные нюансы и ловушки (2026)

| Проблема / Вопрос                                 | Ответ / Рекомендация                                      |
|---------------------------------------------------|------------------------------------------------------------|
| `split` возвращает `Substring`, а не `[String]`   | Да → используйте `.map(String.init)` если нужен `[String]` |
| Почему `split(separator: ",")` → `Substring`?     | Экономия памяти, нет лишних аллокаций                      |
| `split` ленивый?                                  | Да — до первого обращения к элементам ничего не вычисляется |
| Как получить `[String]` быстро?                   | `split(...).map(String.init)` или `components(separatedBy:)` |
| `components(separatedBy:)` vs `split`?            | `components` → всегда `[String]`, аллоцирует всё сразу     |
| Можно ли `split` по регулярке?                    | Нет → используйте `NSRegularExpression` или `Regex`        |

## Когда использовать что

| Задача                                      | Рекомендуемый метод                          |
|---------------------------------------------|----------------------------------------------|
| Нужно быстро разбить и получить Substring   | `split(separator:)` или `split(where:)`      |
| Нужен массив обычных String                 | `components(separatedBy:)` или `split().map(String.init)` |
| Разбиение по сложному условию               | `split(whereSeparator:)`                     |
| CSV / TSV с пустыми полями                  | `split(…, omittingEmptySubsequences: false)` |
| Ограничить количество частей (лог, путь)    | `maxSplits: N`                               |
| Очень большие строки (>1–10 МБ)             | `split` (ленивый) предпочтительнее            |

## Итог — современные рекомендации

- Хотите **максимальную эффективность и ленивость** → `split(separator:)` / `split(where:)`
- Нужен **простой массив строк** и строка небольшая → `components(separatedBy:)`
- Парсите **CSV, логи, пути, параметры URL** → почти всегда `split` с нужными параметрами
- Работаете с **очень большими текстами** → `split` + ленивая обработка

`split` — это один из тех методов, которые кажутся простыми, но при правильном использовании сильно экономят память и упрощают парсинг.
