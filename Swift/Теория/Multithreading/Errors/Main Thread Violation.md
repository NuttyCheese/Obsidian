### 1. Что такое Main Thread Violation

**Main Thread Violation** — это ситуация, когда **операции с пользовательским интерфейсом** ([[UIKit]], [[SwiftUI]], [[AppKit]] и т.д.) выполняются **не в главном потоке** (Main Thread / UI Thread).

**Главный поток** — это **единственный поток**, который имеет право:

- изменять свойства UI-элементов (`label.text`, `view.backgroundColor`, `tableView.reloadData` и т.д.)  
- вызывать методы, связанные с UI (`present`, `pushViewController`, `setNeedsLayout`, `layoutIfNeeded`)  
- запускать анимации (`UIView.animate`, `CAAnimation`)  
- работать с Auto Layout в некоторых случаях

**Любое нарушение этого правила** может привести к:

- визуальным багам (не обновился текст, пропали/появились элементы)  
- зависанию UI  
- случайным крашам (`EXC_BAD_ACCESS`, `NSInternalInconsistencyException`)  
- некорректному поведению анимаций  
- трудно воспроизводимым багам (чаще на реальных устройствах, чем в симуляторе)

**Коротко и по-человечески**:  
«Если ты трогаешь кнопки, лейблы, таблицы, коллекции, навигацию или анимации — делай это **только** в главном потоке. Всё остальное — это нарушение.»

### 2. Самые частые сценарии Main Thread Violation в 2026

| №   | Сценарий                                             | Как проявляется                                     | Частота |
| --- | ---------------------------------------------------- | --------------------------------------------------- | ------- |
| 1   | Сетевой запрос → обновление UI из completion         | Текст не появляется, таблица не перезагружается     | ★★★★★   |
| 2   | DispatchQueue.global().async → UI                    | Баги только в релизе, симулятор может работать      | ★★★★★   |
| 3   | Task.detached {} → UI без @MainActor                 | Swift 6 выдаёт ошибку, [[Swift]] 5 — молча ломается | ★★★★☆   |
| 4   | [[Core Data]] / [[Realm]] save → UI update           | Краш или визуальные артефакты                       | ★★★★☆   |
| 5   | [[NotificationCenter]] → UI update                   | Случайные баги при быстром переключении экранов     | ★★★☆☆   |
| 6   | [[Timer]] / [[CADisplayLink]] → UI без Dispatch.main | Пропадают кадры, UI «дергается»                     | ★★★☆☆   |
| 7   | [[Closure]] из сторонней библиотеки → UI             | Библиотека вызывает closure в фоне                  | ★★★☆☆   |

### 3. Классические примеры нарушений (и как они выглядят)

#### Пример 1 — Сетевой запрос (самый частый случай)

```swift
// Плохо — нарушение Main Thread
func loadUser() {
    URLSession.shared.dataTask(with: url) { data, _, _ in
        let user = try! JSONDecoder().decode(User.self, from: data!)
        nameLabel.text = user.name           // ← Main Thread Violation!
        tableView.reloadData()               // ← Violation!
    }.resume()
}
```

**Проявление**: текст может не появиться, таблица не обновится, или приложение крашнется.

#### Пример 2 — Task.detached без @MainActor (Swift Concurrency)

```swift
// Плохо — в Swift 6 это ошибка компиляции
func loadPosts() {
    Task.detached {
        let posts = try await fetchPosts()
        postsTableView.reloadData()  // ← Violation!
    }
}
```

**В Swift 6**: компилятор выдаст ошибку:  
`Sending 'postsTableView' risks causing data races`

#### Пример 3 — NotificationCenter без Dispatch.main

```swift
// Плохо
NotificationCenter.default.addObserver(forName: .userDidLogin, object: nil, queue: nil) { _ in
    profileImageView.image = newImage  // ← Violation!
}
```

### 4. Все современные способы исправления (2026 стандарт)

| Способ                                | Уровень безопасности | Сложность | Когда использовать в 2026  | Примечание          |
| ------------------------------------- | -------------------- | --------- | -------------------------- | ------------------- |
| **DispatchQueue.main.async**          | ★★★★★                | ★★☆☆☆     | Legacy-код, [[GCD]]        | Классика            |
| **@MainActor**                        | ★★★★★                | ★★☆☆☆     | [[SwiftUI]], ViewModel, UI | Рекомендуется Apple |
| **await MainActor.run**               | ★★★★★                | ★★★☆☆     | Внутри non-MainActor       | Для async кода      |
| **Task { @MainActor in ... }**        | ★★★★★                | ★★☆☆☆     | Запуск UI-обновлений       | Удобно              |
| **@globalActor** (например, @UIActor) | ★★★★★                | ★★★★☆     | Глобальные UI-сервисы      | Редко               |
| **RunLoop.main.perform**              | ★★★★☆                | ★★★☆☆     | Очень старый код           | Legacy              |


### 5. Самые безопасные конструкции 2026 (рекомендуемые Apple)

#### Вариант 1 — @MainActor (самый рекомендуемый в 2026)

```swift
@MainActor
class ProfileViewModel: ObservableObject {
    @Published var user: User?
    
    func load() async {
        let fetched = try await fetchUser()
        user = fetched  // безопасно — @MainActor
    }
}
```

#### Вариант 2 — Task { @MainActor in ... }

```swift
func loadImage(url: URL) async {
    let (data, _) = try await URLSession.shared.data(from: url)
    
    Task { @MainActor in
        imageView.image = UIImage(data: data)
    }
}
```

#### Вариант 3 — await MainActor.run

```swift
func processData() async {
    let processed = await heavyComputation()
    
    await MainActor.run {
        label.text = processed.description
        tableView.reloadData()
    }
}
```

#### Вариант 4 — DispatchQueue.main.async (legacy, но всё ещё работает)

```swift
func updateUI(text: String) {
    DispatchQueue.main.async {
        label.text = text
    }
}
```

### 6. Как найти Main Thread Violation в 2026

| Инструмент / Способ                   | Что ловит                               | Где включается                   | Эффективность |
| ------------------------------------- | --------------------------------------- | -------------------------------- | ------------- |
| **Main Thread Checker**               | [[UIKit]]/[[SwiftUI]] вызовы не из main | [[Xcode]] → Scheme → Diagnostics | ★★★★★         |
| **Swift 6 Strict Concurrency**        | Передача UI-элементов между потоками    | Build Settings → Swift Compiler  | ★★★★★         |
| **Thread Sanitizer**                  | [[Data Race]] + некоторые UI-violations | Xcode → Diagnostics              | ★★★★☆         |
| **Instruments → Main Thread Checker** | UI-вызовы из фона                       | Instruments                      | ★★★★☆         |
| **Xcode Runtime Issues**              | Нарушения в [[runtime]]                 | Debug navigator → Runtime Issues | ★★★★☆         |

### 7. Лучшие практики 2026 (Swift 6+)

- **Переходите на Swift 6** — strict concurrency checking ловит почти все Main Thread Violation  
- **@MainActor** — всё, что касается UI, SwiftUI, ViewModel  
- **actor** — для бизнес-логики и данных  
- **Sendable** — всё, что передаётся между акторами / потоками  
- **Никогда** не вызывайте UI-код из `Task.detached`, `DispatchQueue.global()`, `URLSession` completion  
- **Все сетевые/тяжёлые операции** — в `Task` или `actor`, UI-обновления — через `@MainActor`  
- **Для legacy-кода** — оборачивайте в `@MainActor` или `DispatchQueue.main.async`  
- **В тестах** — используйте `await MainActor.run` или `@MainActor` в тестовых классах  
- **Избегайте** `DispatchQueue.main.sync` — это почти всегда deadlock

**Короткий девиз 2026**:
> «Main Thread Violation — это когда ты трогаешь UI не из главного потока.  
> В 2026 году ответ один: @MainActor + Swift 6 strict concurrency.  
> DispatchQueue.main.async — это уже legacy. Забудьте про него в новом коде.»
