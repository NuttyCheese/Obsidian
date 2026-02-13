### 1. Что такое await и зачем оно нужно

**`await`** — это **инструкция приостановки** (suspension point), которая говорит:

> «Здесь я жду завершения асинхронной операции ([[async]]-функции, [[async let]], [[Task]].sleep, [[actor]]-метода и т.д.).  
> Пока я жду — **не блокируй поток**, отпусти его для других задач».

Это ключевое слово делает конкурентный код **структурированным**, **читаемым** и **безопасным** — вместо [[callback]]-hell или [[DispatchQueue]].[[main]].[[async]] мы пишем линейный код.

**Самое важное в 2026 году**:
- `await` — **единственный способ** получить результат от асинхронной функции  
- без `await` компилятор **не пропустит** вызов `async`-метода  
- `await` **не блокирует поток** — это **кооперативная многозадачность** (cooperative multitasking)

### 2. Где и когда нужно писать await (таблица 2026)

| Тип операции                              | Нужно ли await? | Пример правильного вызова                     | Что будет, если забыть await |
|-------------------------------------------|-----------------|-----------------------------------------------|-------------------------------|
| Обычная `async` функция                   | Да              | `let data = await fetchData()`                | Ошибка компиляции             |
| Метод `actor`-а                           | Да              | `await counter.increment()`                   | Ошибка компиляции             |
| `async let` переменная                    | Да (при чтении) | `let user = await userTask`                   | Предупреждение / утечка результата |
| `Task.sleep` / `Task.yield`               | Да              | `try await Task.sleep(for: .seconds(1))`      | Ошибка компиляции             |
| `MainActor.run { ... }`                   | Да              | `await MainActor.run { label.text = "Hi" }`   | Ошибка компиляции             |
| `withCheckedContinuation` / `withTaskGroup` | Да (на результатах) | `let result = await group.next()`          | Ошибка компиляции             |

**Золотое правило**:
> Если функция помечена `async` или возвращает [[AsyncSequence]] / `AsyncThrowingSequence` → **всегда** используй `await` при получении результата.

### 3. Самые популярные и правильные шаблоны await 2026 года

#### Шаблон 1 — Простейший await (самый частый)

```swift
func fetchUser() async throws -> User {
    // ... сетевой запрос ...
}

Task {
    do {
        let user = try await fetchUser()
        await MainActor.run {
            nameLabel.text = user.name
        }
    } catch {
        print("Ошибка:", error)
    }
}
```

#### Шаблон 2 — await внутри [[@MainActor]] (очень частый в UI)

```swift
@MainActor
class ProfileViewModel: ObservableObject {
    @Published var profile: Profile?
    @Published var isLoading = false

    func load() async {
        isLoading = true
        defer { isLoading = false }  // defer работает и в async!

        let fetched = try? await api.fetchProfile()
        profile = fetched
    }
}
```

#### Шаблон 3 — await + async let (параллельный + последовательный)

```swift
func fetchUser()    async throws -> User    { ... }
func fetchPosts()   async throws -> [Post]  { ... }
func process(posts: [Post]) async throws -> [ProcessedPost] { ... }

Task {
    async let userTask  = fetchUser()
    async let postsTask = fetchPosts()

    let user = try await userTask
    
    // ждём только posts, user уже готов
    let rawPosts = try await postsTask
    
    let processed = try await process(posts: rawPosts)
    
    await MainActor.run {
        self.user = user
        self.posts = processed
    }
}
```

#### Шаблон 4 — await в цикле (обработка стрима)

```swift
let stream = AsyncStream<Int> { continuation in
    Task {
        for i in 1...10 {
            continuation.yield(i)
            try? await Task.sleep(for: .seconds(1))
        }
        continuation.finish()
    }
}

Task {
    for await number in stream {
        print("Получено:", number)
        // можно безопасно обновлять UI
        await MainActor.run {
            label.text = "Число: \(number)"
        }
    }
}
```

### 4. Типичные ошибки и ловушки 2026 года

| Ошибка                                      | Последствия                              | Как избежать |
|---------------------------------------------|------------------------------------------|--------------|
| Забыть `await` перед вызовом async-функции  | Ошибка компиляции                        | Компилятор напомнит |
| Делать долгие синхронные операции внутри @MainActor | UI freeze (ANR)                          | Тяжёлое — в `Task.detached` или обычный `actor` |
| Вызывать actor-метод без `await`            | Ошибка компиляции                        | Всегда `await actor.method()` |
| Использовать `await` в не-async контексте   | Ошибка компиляции                        | Оборачивать в `Task { await ... }` |
| Забыть `try` перед `async throws`           | Ошибка компиляции                        | `try await` всегда |
| Делать последовательные await вместо async let | Медленный код                            | Используй `async let` для независимых задач |

### 5. await vs другие механизмы ожидания (2026 сравнение)

| Механизм                  | Блокирует поток? | Поддержка throws | Параллелизм | Рекомендация 2026 | Когда использовать |
|---------------------------|------------------|-------------------|-------------|-------------------|---------------------|
| `await func()`            | Нет              | Да                | Нет         | Основной          | Последовательные вызовы |
| `await async let`         | Нет              | Да                | Да          | Золотой стандарт  | 2–6 параллельных задач |
| `await withTaskGroup`     | Нет              | Да                | Да          | Масштабируемость  | Много задач         |
| `DispatchQueue.main.async`| Да (если sync)   | Нет               | Нет         | Legacy            | Старый код          |
| `RunLoop.current.run()`   | Да               | Нет               | Нет         | Legacy            | Очень старый код    |

### 6. Лучшие практики 2026 года

- **Всегда** пиши `await` перед любым `async`-вызовом — это закон  
- **Используй `async let`** для независимых параллельных задач (2–6)  
- **Тяжёлое** (сеть, диск, вычисления) — выноси из `@MainActor`  
- **Ошибки** — всегда `try await` и `do-try-catch`  
- **Отмена** — проверяй `Task.isCancelled` внутри долгих операций  
- **Swift 6 strict concurrency** — включай полную проверку — она ловит забытые `await` и unsafe захваты  
- **Мониторинг** — Instruments → Swift Tasks — смотри suspension points и время ожидания

**Короткий девиз 2026**:
> «await — это когда ты говоришь: «я подожду здесь, но не блокируй поток — пусть другие задачи работают».  
> В 2026 году почти весь асинхронный код в Swift держится именно на await.»
