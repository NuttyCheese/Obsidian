**`do-catch`** — это основная конструкция в Swift для **безопасной обработки ошибок**, которые могут быть выброшены функциями с ключевым словом [[throws]].

Она позволяет:
- Выполнить потенциально опасный код внутри `do`
- Перехватить и обработать любую ошибку в блоках `catch`
- Отделить **основную логику** от **обработки ошибок**

Это **единственный правильный** способ работать с `throws` в современном Swift (2026 год).

### 1. Полный синтаксис и все варианты (самые актуальные в 2026)

#### Вариант 1: Базовый do-catch (самый частый)

```swift
do {
    try riskyOperation()
    print("Всё прошло успешно")
} catch {
    print("Произошла ошибка: \(error.localizedDescription)")
}
```

- `error` — это **любая** ошибка, соответствующая протоколу [[Error]]
- Блок `catch` ловит **всё**, что не было обработано выше

#### Вариант 2: Ловля конкретных ошибок (рекомендуемый стиль)

```swift
enum AuthError: Error {
    case wrongCredentials
    case networkFailure
    case serverError(statusCode: Int)
}

do {
    try authenticateUser()
} catch AuthError.wrongCredentials {
    showAlert("Неверный логин или пароль")
} catch AuthError.networkFailure {
    showAlert("Нет соединения с интернетом")
} catch AuthError.serverError(let code) {
    showAlert("Ошибка сервера: \(code)")
} catch {
    showAlert("Неизвестная ошибка: \(error)")
}
```

**Золотое правило 2026**:
- Конкретные `catch` → сверху
- Универсальный `catch` → **всегда последний**

#### Вариант 3: do-catch + [[async]]/[[await]] (самый частый сценарий сейчас)

```swift
Task {
    do {
        let user = try await api.fetchCurrentUser()
        await MainActor.run {
            updateUI(with: user)
        }
    } catch NetworkError.timeout {
        await showTimeoutAlert()
    } catch APIError.unauthorized {
        await navigateToLogin()
    } catch {
        await showGenericError(error.localizedDescription)
    }
}
```

#### Вариант 4: try? и [[try]]! — когда catch не нужен

```swift
// try? — ошибка → nil, без throw
let user = try? JSONDecoder().decode(User.self, from: jsonData)

// try! — если ошибка → краш (fatal error)
let config = try! PropertyListDecoder().decode(Config.self, from: data)
```

**Когда использовать**:
- `try?` — когда ошибка не критична и обработка не нужна
- `try!` — **только** в тестах или когда 100% уверен (очень редко)

### 2. Полный разбор всех catch-вариантов (2026 топ)

```swift
do {
    try complexOperation()
} catch let error as NSError where error.code == -1009 {
    // Специфическая ошибка сети
    print("Нет соединения")
} catch let error as DecodingError {
    // Ошибки декодирования JSON
    switch error {
    case .keyNotFound(let key, _):
        print("Отсутствует ключ: \(key.stringValue)")
    default:
        print("Ошибка декодирования: \(error)")
    }
} catch let error as CustomError where error.isRecoverable {
    // Восстанавливаемая ошибка
    retryOperation()
} catch {
    // Всё остальное
    print("Необработанная ошибка: \(error)")
}
```

### 3. Лучшие практики do-catch в Swift 2026

- **Лови конкретные ошибки первыми** — порядок catch имеет значение
- **Всегда оставляй универсальный catch последним** — для неизвестных ошибок
- **Используй `localizedDescription`** для показа пользователю
- **Логируй неизвестные ошибки** — Crashlytics, Sentry, print или os_log
- **В async** — оборачивай в `Task {}` и обновляй UI через `await MainActor.run`
- **Не используй try! в production** — лучше `try?` + обработка nil
- **Swift 6 strict concurrency** — весь блок `do-try-catch` должен быть на одном акторе
- **Документируйте** — пиши комментарий «do-catch — обработка ошибок авторизации»

**Короткий девиз 2026**:
> `do-catch` — это когда ты говоришь: «попробуй, а если что-то пойдёт не так — я готов».  
> В 2026 году:  
> - конкретные ошибки → сверху  
> - универсальный `catch` → снизу  
> - `try?` — когда ошибка не важна  
> - `try!` — почти никогда  
> Это **единственный безопасный** способ работать с `throws` и `async throws`.
