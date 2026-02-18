**Closure** в Swift — это **самостоятельный блок кода**, который можно передавать как значение, присваивать переменной, возвращать из функции или использовать как аргумент.  
По сути, это **анонимная функция**, которая может **захватывать** (capture) переменные и константы из окружающего контекста.

В 2026 году closure — один из самых часто используемых инструментов Swift: они лежат в основе **async/await**, **Combine**, **SwiftUI**, **higher-order functions** (`map`, `filter`, `reduce`), **completion handlers** и многого другого.

### 1. Зачем нужны closure (реальные сценарии 2026)

| Сценарий                                      | Почему closure идеален                                   | Пример использования |
|-----------------------------------------------|----------------------------------------------------------|----------------------|
| Completion handler (callback)                 | Асинхронная операция сообщает о завершении               | `URLSession`, `UIView.animate`, `CLLocationManager` |
| Higher-order functions                        | Передача логики обработки коллекций                      | `array.map { $0 * 2 }`, `filter { $0 > 0 }` |
| SwiftUI / Combine                             | `@State`, `@Binding`, `Publisher.sink`, `onReceive`      | `Button(action: { ... }) { Text("Tap") }` |
| Async/await обёртки                           | Преобразование старого callback API в async              | `withCheckedContinuation`, `withTaskGroup` |
| Захват состояния (stateful closure)           | Замыкание хранит переменные между вызовами               | `makeCounter()`, `lazy var expensive` |
| Event handling                                | Обработка жестов, уведомлений, KVO                       | `addAction(UIAction { ... })`, `NotificationCenter` |

### 2. Полный синтаксис closure (все варианты)

#### Вариант 1: Полный синтаксис (самый подробный)

```swift
let add: (Int, Int) -> Int = { (a: Int, b: Int) -> Int in
    return a + b
}
print(add(3, 5)) // 8
```

#### Вариант 2: Упрощённый (самый частый)

```swift
let multiply = { a, b in a * b }
print(multiply(4, 6)) // 24
```

#### Вариант 3: Shorthand argument names (`$0`, `$1`)

```swift
let numbers = [1, 2, 3, 4]
let doubled = numbers.map { $0 * 2 }          // [2, 4, 6, 8]
let even = numbers.filter { $0 % 2 == 0 }     // [2, 4]
let sum = numbers.reduce(0, +)                // 10
```

#### Вариант 4: Trailing closure (самый читаемый)

```swift
UIView.animate(withDuration: 0.3) {
    self.view.alpha = 0.5
} completion: { _ in
    self.view.removeFromSuperview()
}
```

#### Вариант 5: Escaping closure + capture list (самый важный в 2026)

```swift
class ViewModel {
    var data: String?
    
    func fetch() {
        URLSession.shared.dataTask(with: url) { [weak self] data, _, error in
            guard let self else { return }
            DispatchQueue.main.async {
                self.data = String(data: data ?? Data(), encoding: .utf8)
            }
        }.resume()
    }
}
```

### 3. Capture List — сердце безопасности closure

```swift
// Опасно — retain cycle
Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { timer in
    self.counter += 1  // self удерживается closure → closure удерживает timer → timer удерживает self
}

// Безопасно — 2026 стандарт
Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] timer in
    guard let self else {
        timer.invalidate()
        return
    }
    self.counter += 1
}
```

**Золотые правила capture list 2026**:
- `[weak self]` — в **любом escaping** closure (async, completion, Timer, Notification)
- `guard let self else { return }` — сразу после входа в closure
- `[unowned self]` — **только** если уверен, что `self` живее closure (редко!)
- `[value]` — захват по значению (snapshot) для констант, параметров, immutable данных
- `[weak delegate, weak dataSource]` — для нескольких объектов

### 4. Closure vs async/await (2026 реальность)

| Ситуация                                      | Старый стиль (closure)                               | Новый стиль (async/await)                            | Рекомендация |
|-----------------------------------------------|-------------------------------------------------------|------------------------------------------------------|--------------|
| Сетевой запрос                                | `dataTask` + completion                               | `try await URLSession.shared.data(from:)`            | async/await |
| Анимация                                      | `animate` + completion                                | `withAnimation { ... }` + `Task`                     | withAnimation |
| Таймер                                        | `Timer` + escaping closure                            | `Task { try await Task.sleep(...) }`                 | Task + sleep |
| Несколько операций                            | Callback hell (вложенные closure)                     | `async let`, `TaskGroup`, `actor`                    | Structured Concurrency |
| Legacy API                                    | Completion handler                                    | `withCheckedContinuation`, `withCheckedThrowingContinuation` | Обёртка в async |

### 5. Лучшие практики closure в Swift 2026

- **Всегда** `[weak self]` + `guard let self` в escaping closure  
- **Используй trailing closure** — читаемость кода растёт на 200%  
- **Shorthand `$0`, `$1`** — в простых `map`/`filter`, но с именами в сложных closure  
- **Избегай capture list в non-escaping** — компилятор сам подскажет, если нужен  
- **Для UI** — все обновления через `await MainActor.run` или `@MainActor`  
- **Swift 6 strict concurrency** — closure должен захватывать `self` явно, иначе ошибка  
- **Документируйте** — пиши комментарий «[weak self] — escaping closure для загрузки данных»

**Короткий девиз 2026**:
> Closure — это **анонимная функция**, которую можно передать, вернуть, сохранить и вызвать позже.  
> В 2026 году:  
> - `[weak self]` + `guard let self` — в каждом escaping closure  
> - trailing closure — для читаемости  
> - `async/await` + `Task` — вместо callback hell  
> Это **основа** асинхронного кода, UI и функционального программирования в Swift.

Удачи с чистыми, безопасными и современными замыканиями в твоём коде! 🚀