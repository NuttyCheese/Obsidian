Вот **полное, подробное и максимально актуальное** (на февраль 2026 года) руководство по ключевому слову **`throws`** в Swift — включая все нюансы Swift 6 и строгой конкурентности.

### 1. Что такое `throws` и зачем оно нужно

**`throws`** — это ключевое слово, которое **помечает функцию** (или метод, инициализатор, замыкание), что она **может выбросить ошибку** во время выполнения.

Функция с `throws` имеет **два возможных исхода**:

- **успешное завершение** → возвращает значение (или `Void`)  
- **выброс ошибки** → передаёт управление в ближайший `catch` или крашит задачу

**Главные цели `throws`** (2026):

- явно указывать в сигнатуре, что функция **не гарантирует** успешного результата  
- заставлять вызывающий код **обрабатывать ошибки** (через `try`, `do-catch`, `try?`, `try!`)  
- делать ошибки **частью типа** функции — компилятор **не даст** их игнорировать  
- поддерживать **чистый и предсказуемый** контроль ошибок вместо возврата `nil` / `Optional` / специальных кодов

**Коротко**:
> `throws` = «эта функция может сломаться — будь готов поймать ошибку».

### 2. Основные варианты использования throws (таблица 2026)

| Вариант                                   | Синтаксис                              | Когда использовать                          | Как вызывать                     |
|-------------------------------------------|----------------------------------------|---------------------------------------------|-----------------------------------|
| `throws`                                  | `func f() throws -> T`                 | Функция может выбросить ошибку              | `try f()`                         |
| `async throws`                            | `func f() async throws -> T`           | Асинхронная функция с ошибками              | `try await f()`                   |
| `rethrows`                                | `func f(_ closure: () throws -> Void) rethrows` | Функция выбрасывает ошибку только если closure выбросил | `try f { ... }`                   |
| `throws` в замыкании                      | `@escaping (Result<T, Error>) -> Void` → `throws` внутри | Legacy → modern bridge                      | `try await withCheckedThrowingContinuation` |
| `throws` + `never` (редко)                | `func f() throws -> Never`             | Функция всегда выбрасывает ошибку           | `try f()` → никогда не возвращает |

### 3. Самые популярные шаблоны throws 2026 года

#### Шаблон 1 — Классический do-try-catch (самый надёжный)

```swift
enum NetworkError: Error {
    case offline
    case timeout
    case badResponse(Int)
}

func fetchProfile() throws -> Profile {
    // ... симуляция сети ...
    if Bool.random() { throw NetworkError.offline }
    if Bool.random() { throw NetworkError.timeout }
    return Profile(name: "Alex")
}

do {
    let profile = try fetchProfile()
    print("Профиль:", profile.name)
} catch NetworkError.offline {
    print("Нет интернета")
} catch NetworkError.timeout {
    print("Таймаут")
} catch NetworkError.badResponse(let code) {
    print("Ошибка сервера:", code)
} catch {
    print("Неизвестная ошибка:", error)
}
```

#### Шаблон 2 — try? — безопасный Optional (очень частый)

```swift
let profile = try? fetchProfile()
if let profile {
    print("Успех:", profile.name)
} else {
    print("Не удалось загрузить профиль")
}
```

#### Шаблон 3 — try! — когда уверен на 100% (опасно, но иногда оправдано)

```swift
let config = try! loadConfigFromDisk()  // если файл точно есть
print(config.apiKey)
```

**Внимание**: `try!` → **краш приложения** при любой ошибке. Используй **только** когда ошибка невозможна.

#### Шаблон 4 — async throws + try await (самый частый в 2026)

```swift
func fetchUser(id: Int) async throws -> User {
    let url = URL(string: "https://api.com/users/\(id)")!
    let (data, response) = try await URLSession.shared.data(from: url)
    
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw URLError(.badServerResponse)
    }
    
    return try JSONDecoder().decode(User.self, from: data)
}

Task {
    do {
        let user = try await fetchUser(id: 42)
        await MainActor.run {
            nameLabel.text = user.name
        }
    } catch {
        await MainActor.run {
            errorLabel.text = "Ошибка: \(error.localizedDescription)"
        }
    }
}
```

#### Шаблон 5 — rethrows (передача ошибки из замыкания)

```swift
func withRetry<T>(_ operation: () throws -> T, maxAttempts: Int = 3) rethrows -> T {
    var lastError: Error?
    
    for attempt in 1...maxAttempts {
        do {
            return try operation()
        } catch {
            lastError = error
            print("Попытка \(attempt) провалилась")
        }
    }
    
    throw lastError ?? NSError(domain: "Unknown", code: -1)
}

// Использование
try withRetry {
    try riskyOperation()
}
```

### 4. Типичные ошибки и ловушки throws 2026 года

| Ошибка                                      | Последствия                              | Как избежать |
|---------------------------------------------|------------------------------------------|--------------|
| Забыть `try` перед вызовом throws-функции   | Ошибка компиляции                        | Компилятор напомнит |
| Использовать `try?` и не проверять Optional | Игнорирование ошибок → silent failure    | Всегда проверять `if let` / `guard let` |
| `try!` в production-коде                    | Краш приложения при любой ошибке         | Только в тестах / когда ошибка невозможна |
| Не обрабатывать конкретные ошибки           | Общая обработка → плохой UX              | Используй `catch SpecificError { ... }` |
| throws в методе, который не должен падать   | Неожиданные краши                        | Верни `Result` / Optional вместо throws |
| throws + async без try await                | Ошибка компиляции                        | Всегда `try await` для `async throws` |

### 5. throws vs другие способы обработки ошибок (2026 сравнение)

| Механизм                  | Проверка компилятором | Читаемость | Обработка ошибок | Рекомендация 2026 | Когда использовать |
|---------------------------|------------------------|------------|-------------------|-------------------|---------------------|
| `throws` / `try` / `do-catch` | Полная                 | Высокая    | Полная            | Основной выбор    | Почти всё           |
| `Result<T, Error>`        | Нет                    | Средняя    | Ручная            | Legacy / Combine  | Старые API          |
| `Optional` + `try?`       | Частично               | Высокая    | Частичная         | Простые случаи    | Когда ошибка не критична |
| `fatalError` / `preconditionFailure` | Нет              | Низкая     | Нет               | Отладка           | Невозможные случаи  |
| `Never` (как тип возврата)| Полная                 | Высокая    | Нет               | Редко             | Функции, которые всегда крашат |

**Вывод 2026**:
- **`throws` + `try` / `do-catch`** — **основной и рекомендуемый** способ обработки ошибок в Swift  
- `Result` — только для legacy / Combine / старых API  
- `Optional` — для случаев, когда ошибка **не критична**  
- В Swift 6 с полной проверкой конкурентности `throws` стал **ещё строже** и **ещё полезнее**

### 6. Лучшие практики 2026 года

- **Всегда** используй конкретные типы ошибок (`enum MyError: Error`)  
- **Обрабатывай** конкретные ошибки через `catch SpecificError { ... }`  
- **Используй `try?`** только когда ошибка **не критична** и результат — Optional  
- **Никогда** не используй `try!` в production-коде  
- **Для async** — всегда `try await`  
- **Rethrows** — используй для функций-обёрток над замыканиями  
- **Swift 6 strict concurrency** — включай полную проверку — она ловит забытые `try`  
- **Тестирование** — пиши тесты на каждый `catch`-случай  
- **Документируй** — пиши в документации все возможные `throws`

**Короткий девиз 2026**:
> «throws — это когда ты честно говоришь: «я могу сломаться, будь готов».  
> В 2026 году это **основной** способ работы с ошибками в Swift — чистый, проверяемый и безопасный.»

Удачи с надёжным, читаемым и современным обработчиком ошибок в Swift! 🛡️