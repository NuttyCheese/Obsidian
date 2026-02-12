#memory_control #Swift #error 
## 📘 Определение
**Runtime Error** — это ошибка, которая возникает **во время выполнения программы** (run-time), а не на этапе компиляции.  
Компилятор [[Swift]] не может её предсказать, поэтому программа падает (crash) с сообщением в консоли и стек-трейсом.

### Самые частые причины Runtime Error в [[iOS]]/Swift (2026)

| №   | Ошибка                                              | Типичное сообщение в Xcode                              | Как избежать / исправить                                |
| --- | --------------------------------------------------- | ------------------------------------------------------- | ------------------------------------------------------- |
| 1   | **Force unwrap [[nil]]** (`!`)                      | `unexpectedly found nil while unwrapping an Optional`   | Используй [[if let]], [[guard let]], `??`               |
| 2   | **Index out of range**                              | `Index out of range`                                    | Проверяй `indices.contains(index)` или `safe subscript` |
| 3   | **Division by zero**                                | `Fatal error: Division by zero`                         | Проверяй делитель перед операцией                       |
| 4   | **Invalid type cast** (`as!`)                       | `Could not cast value of type '…' to '…'`               | Используй `as?` + `if let`                              |
| 5   | **Fatal error / preconditionFailure**               | Сообщение из `fatalError` или `precondition`            | Используй `guard` / `assert` в debug                    |
| 6   | **Threading violation** (UI с background)           | `EXC_BAD_ACCESS` или `NSInternalInconsistencyException` | Все UI-обновления — на main thread                      |
| 7   | **[[Array]] / [[Dictionary]] [[Swift/Теория/Ошибки и варнинги/Рантайм ошибки/force unwrap]]** | `unexpectedly found nil`                                | `if let value = dict[key]`                              |

### Примеры и безопасные альтернативы

```swift
// 1. Опасно (crash при nil)
let name: String? = nil
print(name!)  // → crash

// Безопасно
if let name = name {
    print(name)
} else {
    print("Имя отсутствует")
}

// 2. Опасно (index out of range)
let arr = [1, 2, 3]
print(arr[5])  // → crash

// Безопасно
if arr.indices.contains(5) {
    print(arr[5])
} else {
    print("Индекс вне диапазона")
}

// 3. Опасно (as!)
let value: Any = "42"
let num = value as! Int  // → crash

// Безопасно
if let num = value as? Int {
    print(num)
} else {
    print("Не число")
}
```

### Полезные инструменты для поиска и отладки (2026)

- **Xcode → Exception Breakpoint** — останавливает на строке с Runtime Error
- **Instruments → Allocations / Leaks** — показывает, где происходит crash
- **Memory Graph Debugger** — выявляет [[retain cycle]] перед crash
- **Thread Sanitizer** — ловит race conditions и UI с background
- **Swift Strict Concurrency Checking** (в Swift 6) — предотвращает многие runtime-ошибки потоков

### Короткие правила предотвращения Runtime Error

- **Никогда** не используй `!` для [[Optional]] (force unwrap)
- Используй `if let`, `guard let`, `??`, `as?`
- Проверяй индексы: `indices.contains`, `safe subscript`
- Для деления → `guard divisor != 0 else { return }`
- `fatalError` / `precondition` — только в debug или для недостижимого кода
- Все UI-обновления — на `DispatchQueue.main`
- Включи **Strict Concurrency Checking** в Swift 6+

**Главное правило 2026**:
> «Если программа может упасть в runtime — значит ты где-то используешь force unwrap, unsafe cast или не проверяешь границы.  
> Пиши безопасный код с `?`, `if let`, `guard`, `as?` — это спасает от 90% Runtime Error.»
