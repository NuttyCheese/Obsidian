### 1. Что такое Writer Monad простыми словами

**Writer Monad** — это монада, которая позволяет **накапливать дополнительную информацию** (лог, метрики, аудит-трейл, сообщения об ошибках, статистику) параллельно с основным вычислением, **не меняя** его сигнатуру и не загрязняя код побочными эффектами.

Представьте, что у вас есть функция, которая возвращает результат, но при этом хочет «по пути» записывать, что она делала:

Без Writer:
```swift
var log: [String] = []

func compute(x: Int) -> Int {
    log.append("Начали с \(x)")
    let y = x * 2
    log.append("Умножили на 2 → \(y)")
    return y
}
```

С Writer:
```swift
let compute = Writer.pure(5)
    .flatMap { x in Writer("Начали с \(x)", x * 2) }
    .flatMap { y in Writer("Умножили на 2 → \(y)", y) }

let (result, log) = compute.run()
// result = 10
// log = ["Начали с 5", "Умножили на 2 → 10"]
```

**Writer Monad** — это пара `(Value, Log)`, где `Log` накапливается по мере выполнения.

### 2. Классическая реализация Writer Monad

```swift
struct Writer<Log, A> {
    let value: A
    let log: Log
    
    init(value: A, log: Log) {
        self.value = value
        self.log = log
    }
    
    // unit / pure / return
    static func pure(_ value: A) -> Writer<Log, A> where Log: Monoid {
        Writer(value: value, log: .empty)
    }
    
    // map — Functor
    func map<B>(_ f: (A) -> B) -> Writer<Log, B> where Log: Monoid {
        Writer(value: f(value), log: log)
    }
    
    // flatMap — Monad
    func flatMap<B>(_ f: (A) -> Writer<Log, B>) -> Writer<Log, B> where Log: Monoid {
        let next = f(value)
        return Writer(value: next.value, log: log.combine(next.log))
    }
    
    // run — просто возвращает пару
    func run() -> (A, Log) {
        (value, log)
    }
}
```

**Важно**: для работы `Writer` требует, чтобы `Log` был **Monoid** (тип с операцией `combine` и нейтральным элементом `empty`).

### 3. Monoid — обязательный спутник Writer

Monoid — это тип, который умеет **комбинировать** значения и имеет **нейтральный элемент**.

Примеры Monoid:

| Тип лога               | Нейтральный элемент (`empty`) | Операция комбинирования (`combine`) |
| ---------------------- | ----------------------------- | ----------------------------------- |
| [[String]]             | `""`                          | `+`                                 |
| `[String]`             | `[]`                          | `+`                                 |
| [[Int]] (сумма)        | `0`                           | `+`                                 |
| [[Set]]`<Element>`     | `[]`                          | `union`                             |
| [[Dictionary]]`<K, V>` | `[:]`                         | `merging`                           |

Пример реализации Monoid для [[String]] и [[Array]]:

```swift
protocol Monoid {
    static var empty: Self { get }
    func combine(_ other: Self) -> Self
}

extension String: Monoid {
    static var empty: String { "" }
    func combine(_ other: String) -> String { self + other }
}

extension Array: Monoid {
    static var empty: [Element] { [] }
    func combine(_ other: [Element]) -> [Element] { self + other }
}
```

### 4. Полные примеры кода

#### Пример 1 — Логирование шагов вычисления

```swift
let step1 = Writer(value: 5, log: "Начали с 5\n")
let step2 = step1.flatMap { x in
    Writer(value: x + 10, log: "Добавили 10 → \(x + 10)\n")
}
let step3 = step2.flatMap { x in
    Writer(value: x * 2, log: "Умножили на 2 → \(x * 2)\n")
}

let (result, log) = step3.run()
print("Результат:", result)  // 30
print("Лог:\n\(log)")
/*
Результат: 30
Лог:
Начали с 5
Добавили 10 → 15
Умножили на 2 → 30
*/
```

#### Пример 2 — Сбор ошибок / предупреждений

```swift
struct ValidationLog {
    var messages: [String] = []
    
    mutating func warning(_ msg: String) {
        messages.append("⚠️ \(msg)")
    }
    
    mutating func error(_ msg: String) {
        messages.append("❌ \(msg)")
    }
}

extension ValidationLog: Monoid {
    static var empty: ValidationLog { ValidationLog() }
    func combine(_ other: ValidationLog) -> ValidationLog {
        var result = self
        result.messages += other.messages
        return result
    }
}

func validateName(_ name: String) -> Writer<ValidationLog, String> {
    var log = ValidationLog()
    if name.isEmpty {
        log.error("Имя не может быть пустым")
    }
    if name.count < 3 {
        log.warning("Имя слишком короткое")
    }
    return Writer(value: name, log: log)
}
```

#### Пример 3 — Асинхронный Writer (2026 стандарт)

```swift
struct AsyncWriter<Log: Monoid, A> {
    let run: () async -> (A, Log)
    
    func flatMap<B>(_ f: @escaping (A) async -> AsyncWriter<Log, B>) async -> AsyncWriter<Log, B> {
        AsyncWriter {
            let (value, log1) = await self.run()
            let (nextValue, log2) = await f(value).run()
            return (nextValue, log1.combine(log2))
        }
    }
}

func asyncFetchUser() async -> AsyncWriter<[String], User> {
    AsyncWriter {
        try? await Task.sleep(nanoseconds: 1_000_000_000)
        return (User(name: "Anna"), ["Запрос к API выполнен"])
    }
}
```

### 5. Реальные сценарии в [[iOS]]-разработке 2026

1. **Логирование бизнес-логики** — собирать аудит-трейл всех шагов  
2. **Валидация формы** — накапливать все ошибки и предупреждения  
3. **Трассировка запросов** — собирать метаданные о каждом шаге [[API]]-цепочки  
4. **Отладка сложных вычислений** — записывать промежуточные результаты  
5. **Аналитика / мониторинг** — считать количество операций, время выполнения  
6. **Тестирование** — проверять, что лог содержит ожидаемые сообщения

### 6. Сравнение Writer Monad с другими монадами

| Монада           | Что инкапсулирует                    | Основной оператор | Когда использовать     |
| ---------------- | ------------------------------------ | ----------------- | ---------------------- |
| **[[Optional]]** | Возможное отсутствие значения        | [[flatMap]]       | Работа с [[nil]]       |
| **Result**       | Успех или ошибка                     | `flatMap`         | Обработка ошибок       |
| **Array**        | Множество значений                   | `flatMap`         | Работа с коллекциями   |
| **Writer**       | Значение + накопленный лог / метрики | `flatMap`         | Логирование, аудит     |
| **Reader**       | Неизменяемое окружение               | `flatMap`         | Dependency Injection   |
| **State**        | Изменяемое состояние                 | `flatMap`         | Управление состоянием  |
| **IO / AsyncIO** | Побочные эффекты                     | `flatMap`         | Работа с внешним миром |

### 7. Лучшие практики 2026

- Используйте Writer с **Monoid**-логом ([[String]], [String], Sum, [[Set]] и т.д.)  
- Для асинхронных операций — **AsyncWriter** с `async throws`  
- **Не выполняйте** `run()` внутри чистых функций — только в оболочке  
- Для тестов — проверяйте как `value`, так и `log`  
- Комбинируйте с **Reader** (ReaderT / WriterT) для передачи зависимостей + лога  
- Для больших приложений — рассмотрите **TCA**, **swift-dependencies** или **Bow** вместо ручного Writer  
- Избегайте глубоких цепочек `flatMap` — разбивайте на маленькие функции

**Короткий девиз 2026**:
> «Writer Monad — это когда ты хочешь не только получить результат, но и рассказать, как ты к нему пришёл.  
> flatMap — это шаг с записью в лог.  
> run() — это момент, когда ты получаешь и результат, и всю историю пути.»
