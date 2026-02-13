### 1. Что такое DispatchGroup и зачем он нужен

**DispatchGroup** — это инструмент [[GCD]], который позволяет:

- отслеживать завершение **нескольких асинхронных задач**  
- выполнить определённый код **только после того**, как **все** задачи завершатся  
- синхронизировать работу между разными очередями (в том числе с главным потоком)

Самые частые сценарии использования в [[iOS]]-приложениях 2026 года:

- Загрузка нескольких изображений / данных → обновить UI после всех  
- Параллельные сетевые запросы → показать результат, когда все готовы  
- Миграция базы данных + загрузка конфига → продолжить запуск приложения  
- Несколько операций записи в [[Core Data]] / [[Realm]] → обновить UI после commit  
- Тестирование асинхронного кода ([[XCTest]] + ожидание группы)

### 2. Основные методы DispatchGroup

| Метод                               | Что делает                                                                 | Когда вызывать                          | Важные замечания |
|-------------------------------------|-----------------------------------------------------------------------------|------------------------------------------|------------------|
| `init()`                            | Создаёт новую пустую группу                                                | Один раз при создании                    | —                |
| `enter()`                           | Увеличивает счётчик незавершённых задач на 1                               | Перед запуском асинхронной задачи        | Каждый enter → должен быть leave |
| `leave()`                           | Уменьшает счётчик незавершённых задач на 1                                 | Когда задача завершена                   | Баланс enter/leave обязателен |
| `notify(queue: DispatchQueue, execute:)` | Выполняет замыкание, когда счётчик достигнет 0                     | После всех enter, но до завершения задач | Можно вызывать несколько раз |
| `wait()`                            | Блокирует текущий поток, пока группа не завершится                         | Только в крайних случаях!                | Опасно — может вызвать deadlock |
| `wait(timeout:)`                    | Блокирует с таймаутом                                                      | Для тестов или защит от зависания        | Возвращает `.timedOut` или `.success` |

### 3. Классические и современные примеры использования (2026)

#### Пример 1 — Классический: загрузка нескольких изображений

```swift
let group = DispatchQueue(label: "image-loading")

let urls = [url1, url2, url3, url4]

var images: [UIImage?] = []

for url in urls {
    group.enter()
    
    URLSession.shared.dataTask(with: url) { data, _, _ in
        if let data, let image = UIImage(data: data) {
            images.append(image)
        }
        group.leave()
    }.resume()
}

group.notify(queue: .main) {
    // Все изображения загружены → обновляем UI
    collectionView.reloadData()
    activityIndicator.stopAnimating()
}
```

#### Пример 2 — Современный ([[Swift Concurrency]] + DispatchGroup)

```swift
func loadAllData() async {
    let group = DispatchGroup()
    
    group.enter()
    Task {
        do {
            let user = try await fetchUser()
            await MainActor.run { self.user = user }
        } catch {
            print(error)
        }
        group.leave()
    }
    
    group.enter()
    Task {
        do {
            let posts = try await fetchPosts()
            await MainActor.run { self.posts = posts }
        } catch {
            print(error)
        }
        group.leave()
    }
    
    await withCheckedContinuation { continuation in
        group.notify(queue: .main) {
            continuation.resume()
        }
    }
    
    print("Все данные загружены")
}
```

#### Пример 3 — DispatchGroup + timeout (защита от зависания)

```swift
let group = DispatchGroup()

group.enter()
someVeryLongTask { result in
    // ...
    group.leave()
}

let timeoutResult = group.wait(timeout: .now() + 10)

switch timeoutResult {
case .success:
    print("Все задачи завершились вовремя")
case .timedOut:
    print("Таймаут — некоторые задачи не успели")
}
```

### 4. Сравнение DispatchGroup с современными альтернативами (2026)

| Механизм                      | Подходит для Swift Concurrency?  | Удобство | Производительность | Когда использовать в 2026            |
| ----------------------------- | -------------------------------- | -------- | ------------------ | ------------------------------------ |
| **DispatchGroup**             | Да (с `withCheckedContinuation`) | ★★★★☆    | ★★★★☆              | Legacy-код, GCD, простые случаи      |
| **[[TaskGroup]]**             | Да (нативно)                     | ★★★★★    | ★★★★★              | Параллельные асинхронные задачи      |
| **[[actor]] + [[Task]]**      | Да                               | ★★★★★    | ★★★★☆              | Состояние + асинхронные операции     |
| **[[async let]]**             | Да                               | ★★★★★    | ★★★★★              | 2–5 независимых задач                |
| **[[withThrowingTaskGroup]]** | Да                               | ★★★★★    | ★★★★★              | Задачи, которые могут бросать ошибки |

**Вывод 2026 года**:
- Если ты пишешь **новый код** — используй **TaskGroup** / **async let** / **actor**  
- **DispatchGroup** остаётся актуальным только для:
  - поддержки старого кода  
  - интеграции с GCD-based библиотеками  
  - очень простых сценариев (2–3 задачи)

### 5. Типичные ошибки с DispatchGroup (и как их избежать)

| Ошибка                                      | Последствия                              | Как избежать |
|---------------------------------------------|------------------------------------------|--------------|
| Забыть `enter()` / `leave()`                | `notify` никогда не сработает           | Всегда парно: enter → leave |
| `enter()` после `notify`                    | `notify` сработает раньше времени       | `notify` вызывать после всех enter |
| `wait()` на текущей очереди                 | Deadlock                                 | Никогда не использовать `wait()` на main или текущей очереди |
| `notify` на той же очереди, где задачи      | Deadlock или бесконечное ожидание        | `notify` на другую очередь (чаще `.main`) |
| Использовать `sync` внутри группы           | Deadlock                                 | Только `async` задачи |

### 6. Лучшие практики 2026 (Swift 6+)

- **Переходите на Swift Concurrency** — `TaskGroup`, `async let`, `actor` почти всегда лучше DispatchGroup  
- **Используйте `defer { leave() }`** — спасает от забытого `leave`  
- **notify всегда на .main** — если нужно обновлять UI  
- **Не используйте `wait()`** в production — только в тестах с таймаутом  
- **Для тестов** — используйте `XCTestExpectation` или `withCheckedContinuation` + `notify`  
- **Для legacy-кода** — постепенно заменяйте DispatchGroup на TaskGroup / actor  
- **Swift 6 strict concurrency** — DispatchGroup совместим, но компилятор может подсвечивать unsafe передачу данных

**Короткий девиз 2026**:
> «DispatchGroup — это когда нужно дождаться нескольких асинхронных задач и сделать что-то после.  
> В 2026 году он всё ещё работает, но почти всегда лучше использовать TaskGroup, async let или actor.  
> enter → leave → notify — святая троица. Забудешь хоть одно — и приложение зависнет.»
