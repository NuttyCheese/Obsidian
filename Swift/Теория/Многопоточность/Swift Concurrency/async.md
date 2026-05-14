Вот **полное, подробное и максимально насыщенное** руководство по ключевому слову **`async`** в Swift (и его взаимодействию с [[await]], [[throws]], [[async let]] и другими механизмами) — актуально на 2026 год (Swift 6+ и строгий режим конкурентности).

### 1. Что такое async и зачем оно нужно

**`async`** — это ключевое слово, которое помечает функцию (или метод, замыкание, инициализатор) как **асинхронную**.

Асинхронная функция:

- может **приостанавливаться** (suspend) в середине выполнения  
- освобождает поток, пока ждёт внешнего события (сеть, диск, таймер, другой Task)  
- возобновляется **на том же actor-е** или потоке, где была приостановлена (если это изолированный контекст)  
- **требует** `await` при вызове (или `@MainActor` / actor-изоляции)

**Коротко и по-человечески**:
> `async` = «эта функция может сказать посреди работы: подожди меня, я сейчас вернусь, а ты пока можешь делать что-то полезное».

Без `async` / [[await]] в [[Swift Concurrency]] почти невозможно писать современный асинхронный код.

### 2. Основные правила async / await (2026)

| Правило                                   | Что это значит                                   | Пример ошибки / правильный код |
|-------------------------------------------|--------------------------------------------------|---------------------------------|
| Функция с `async` **всегда** требует `await` при вызове | Компилятор заставит                                | `fetch()` → ошибка<br>`await fetch()` → ок |
| `async` + `throws` = `async throws`       | Может и приостановиться, и бросить ошибку       | `async throws -> Data`          |
| Внутри `async`-функции можно вызывать другие `async` без `await`, если уже в изоляции | Например, внутри `@MainActor` или actor         | `text = await fetch()` внутри `@MainActor` → можно без `await` в некоторых случаях |
| `async let` — параллельные вызовы         | Запускает несколько `async` параллельно         | `async let a = f1(); async let b = f2()` |
| `Task { ... }` без `@MainActor` — **не наследует** изоляцию | Новый независимый контекст                       | Потеря `@MainActor` → ошибка    |
| `@_inheritActorContext` / `@isolated(any)` | Позволяет сохранить изоляцию в замыкании        | Используется редко, но мощно    |

### 3. Самые популярные шаблоны async 2026 года

#### Шаблон 1 — Самая простая асинхронная функция

```swift
func sayHello() async {
    print("Привет из асинхронной функции")
}

Task {
    await sayHello()
}
```

#### Шаблон 2 — Возврат значения + задержка

```swift
func loadData() async -> String {
    try? await Task.sleep(nanoseconds: 800_000_000) // ~0.8 сек
    return "Данные загружены"
}

Task {
    let data = await loadData()
    print(data)
}
```

#### Шаблон 3 — Обработка ошибок (async [[throws]])

```swift
enum NetworkError: Error {
    case timeout
    case badResponse
}

func fetchUser(id: Int) async throws -> User {
    try await Task.sleep(nanoseconds: 500_000_000)
    if id == 0 { throw NetworkError.badResponse }
    return User(id: id, name: "User \(id)")
}

Task {
    do {
        let user = try await fetchUser(id: 42)
        print("Пользователь:", user.name)
    } catch {
        print("Ошибка:", error)
    }
}
```

#### Шаблон 4 — Параллельные вызовы ([[async let]]) — золотой стандарт 2026

```swift
func fetchUser() async -> User { ... }     // 1.2 сек
func fetchPosts() async -> [Post] { ... }  // 0.9 сек
func fetchComments() async -> [Comment] { ... } // 1.5 сек

Task {
    async let user = fetchUser()
    async let posts = fetchPosts()
    async let comments = fetchComments()

    // все три выполняются параллельно

    let combined = await (user: user, posts: posts, comments: comments)
    print("Всё загружено за ~1.5 сек вместо 3.6 сек")
}
```

#### Шаблон 5 — @MainActor + async (самый частый в UI)

```swift
@MainActor
class ProfileViewModel: ObservableObject {
    @Published var profile: Profile?
    @Published var isLoading = false

    func load() async {
        isLoading = true
        let fetched = try? await api.fetchProfile()
        profile = fetched
        isLoading = false
    }
}

struct ProfileView: View {
    @StateObject private var vm = ProfileViewModel()

    var body: some View {
        VStack {
            if vm.isLoading { ProgressView() }
            Text(vm.profile?.name ?? "Загрузка...")
            Button("Обновить") {
                Task { await vm.load() }
            }
        }
        .task { await vm.load() } // автоматический вызов при появлении
    }
}
```

### 4. Типичные ошибки и ловушки 2026 года

| Ошибка                                      | Последствия                              | Как избежать |
|---------------------------------------------|------------------------------------------|--------------|
| Забыть `await` при вызове async-функции     | Ошибка компиляции                        | Компилятор сам напомнит |
| Вызывать async-функцию без Task / await     | Ошибка компиляции                        | Всегда оборачивать в `Task { await ... }` |
| Делать тяжёлую работу внутри @MainActor     | UI freeze (ANR)                          | Тяжёлое — в обычный `actor` или `Task.detached` |
| Использовать `Task { ... }` внутри actor без изоляции | Потеря контекста → ошибка доступа        | `Task { @MainActor in ... }` или `@_inheritActorContext` (редко) |
| Забыть `try` при `async throws`             | Ошибка компиляции                        | `try await` всегда |
| Последовательные await вместо async let     | Медленный код                            | Используй `async let` для параллелизма |

### 5. async vs другие механизмы конкурентности (2026 сравнение)

| Механизм                  | Последовательность         | Параллелизм | Отмена задач   | Потокобезопасность | Рекомендация 2026         |
| ------------------------- | -------------------------- | ----------- | -------------- | ------------------ | ------------------------- |
| `async` + `await`         | Да                         | Нет         | Через Task     | Нет                | Базовый строительный блок |
| `async let`               | Нет                        | Да          | Да             | Нет                | Параллельные вызовы       |
| `TaskGroup`               | Нет                        | Да          | Да             | Нет                | Много параллельных задач  |
| `actor`                   | Да (изоляция)              | Нет         | Через [[Task]] | Полная             | Изменяемое состояние      |
| `@MainActor`              | Да ([[main\|main thread]]) | Нет         | Через Task     | Полная ([[main]])  | UI и ViewModel            |
| [[DispatchQueue]] + async | Да / Нет                   | Да / Нет    | Ограниченно    | Ручная             | Legacy-код                |

**Вывод 2026**:
- `async` / `await` — **фундамент** всей современной конкурентности в Swift  
- `async let` — для 2–5 параллельных вызовов  
- `TaskGroup` — для произвольного количества параллельных задач  
- `actor` — для защиты изменяемого состояния  
- `@MainActor` — для всего, что касается UI

### 6. Лучшие практики 2026 года

- **Все UI-классы** — помечать `@MainActor`  
- **Всё изменяемое состояние** — хранить в `actor`  
- **Параллельные вызовы** — использовать `async let` (2–5) или `TaskGroup` (много)  
- **Тяжёлая работа** — выносить из `@MainActor` в `Task.detached` или обычный `actor`  
- **Ошибки** — всегда `try await`, `do-try-catch`  
- **Отмена** — используй `Task.cancel()` и проверяй `Task.isCancelled`  
- **Swift 6 strict concurrency** — включай полную проверку — она ловит почти все ошибки  
- **Мониторинг** — Instruments → Swift Tasks + Thread Sanitizer

**Короткий девиз 2026**:
> «async / await — это когда ты говоришь: «я могу подождать, не блокируя поток».  
> В 2026 году почти весь асинхронный код в Swift пишется именно так.»
