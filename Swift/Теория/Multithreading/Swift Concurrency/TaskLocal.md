### 1. Что такое TaskLocal простыми словами

**`TaskLocal`** — это **локальная переменная, уникальная для каждой асинхронной задачи** (`Task`) и её дочерних задач.

Она позволяет:

- хранить данные (контекст), которые **видны только внутри текущей задачи и её потомков**  
- передавать эти данные **между вызовами async-функций без явных параметров**  
- автоматически **наследовать** значение дочерними задачами  
- **переопределять** значение в дочерней задаче без влияния на родительскую

**Коротко**:
> TaskLocal = «невидимый рюкзак», который каждая задача несёт с собой.  
> Дочерние задачи получают копию рюкзака родителя, но могут положить туда своё.

Это **самый чистый способ** передавать контекст (user ID, request ID, trace ID, locale, auth token и т.д.) по всей цепочке асинхронных вызовов.

### 2. Основные правила TaskLocal (2026)

| Правило                            | Что происходит                                     | Пример                                                               |
| ---------------------------------- | -------------------------------------------------- | -------------------------------------------------------------------- |
| Доступ только внутри `Task`        | Вне асинхронного контекста — `nil` или [[default]] | `TaskLocal.currentUser.currentValue` → [[nil]] вне [[Task]]          |
| Наследование дочерними задачами    | Дочерняя задача получает значение родителя         | `Task { Task { print(TaskLocal.x.currentValue) } }` — видит значение |
| Переопределение в дочерней задаче  | Не влияет на родительскую задачу                   | `withValue("child") { ... }` — родитель видит старое значение        |
| Изменение только через `withValue` | Прямое присваивание запрещено                      | `TaskLocal.x = "new"` → ошибка компиляции                            |
| Потокобезопасность                 | Полностью встроенная                               | Можно использовать из любого потока внутри Task                      |
| Можно использовать в `actor`       | Да, но нужно `await` для доступа к состоянию       | `await TaskLocal.x.currentValue` внутри actor                        |
| Default-значение                   | Можно задать через `@TaskLocal(default: ...)`      | `@TaskLocal(default: "guest") static var user`                       |

### 3. Самые популярные шаблоны TaskLocal 2026 года

#### Шаблон 1 — Request ID / Trace ID (самый частый в backend / [[API]])

```swift
@TaskLocal
static var requestID: String?

func handleRequest() async {
    await TaskLocal.requestID.withValue(UUID().uuidString) {
        await processRequest()     // requestID виден во всех вложенных вызовах
        await logMetrics()         // тоже видит requestID
    }
}

func logMetrics() async {
    let rid = TaskLocal.requestID.currentValue ?? "unknown"
    print("Метрики для request \(rid)")
    // отправка в аналитику, Sentry, etc.
}
```

#### Шаблон 2 — Текущий пользователь / Auth Token

```swift
@TaskLocal
static var currentUserID: UUID?

@MainActor
class AuthViewModel: ObservableObject {
    func login(userID: UUID) async {
        await TaskLocal.currentUserID.withValue(userID) {
            await loadProfile()      // видит userID
            await loadFriends()      // тоже видит
        }
    }
}

func loadProfile() async {
    let userID = TaskLocal.currentUserID.currentValue
    // загрузка профиля по userID
}
```

#### Шаблон 3 — Переопределение в дочерней задаче

```swift
@TaskLocal
static var transactionID: String?

Task {
    await TaskLocal.transactionID.withValue("main-tx") {
        Task {
            // дочерняя задача видит "main-tx"
            print(TaskLocal.transactionID.currentValue)  // main-tx
            
            await TaskLocal.transactionID.withValue("sub-tx") {
                // переопределение только внутри этой подзадачи
                print(TaskLocal.transactionID.currentValue)  // sub-tx
            }
            
            // здесь снова "main-tx"
            print(TaskLocal.transactionID.currentValue)  // main-tx
        }
    }
}
```

#### Шаблон 4 — TaskLocal + [[AsyncSequence]] / Stream

```swift
@TaskLocal
static var sessionID: String?

func streamLogs() -> AsyncStream<String> {
    AsyncStream { continuation in
        Task {
            for i in 1...5 {
                let msg = "Log \(i) from session \(TaskLocal.sessionID.currentValue ?? "none")"
                continuation.yield(msg)
                try? await Task.sleep(for: .milliseconds(300))
            }
            continuation.finish()
        }
    }
}

Task {
    await TaskLocal.sessionID.withValue("sess-abc123") {
        for await log in streamLogs() {
            print(log)  // все строки содержат sess-abc123
        }
    }
}
```

### 4. Типичные ошибки и ловушки 2026 года

| Ошибка                                                         | Последствия                       | Как избежать                                                 |
| -------------------------------------------------------------- | --------------------------------- | ------------------------------------------------------------ |
| Прямое присваивание `TaskLocal.x = value`                      | Ошибка компиляции                 | Только `withValue { ... }`                                   |
| Доступ к `TaskLocal.x.currentValue` вне Task                   | `nil` или default                 | Использовать только внутри async-контекста                   |
| Забыть `withValue` и ожидать значение                          | `nil` / default вместо ожидаемого | Всегда оборачивать в `withValue`                             |
| Думать, что изменение в дочерней задаче влияет на родительскую | Нет — значения независимы         | Помнить про копирование при наследовании                     |
| Использовать TaskLocal для глобального состояния               | Нарушение принципа изоляции       | Для глобального — `@globalActor` или [[singleton]] [[actor]] |
| Не учитывать отмену задачи                                     | Значение может остаться «висеть»  | `withValue` автоматически очищается при отмене               |

### 5. TaskLocal vs другие способы передачи контекста (2026 сравнение)

| Механизм             | Передача без явного параметра | Автоматическое наследование | Изоляция | Рекомендация 2026  | Когда использовать                  |
| -------------------- | ----------------------------- | --------------------------- | -------- | ------------------ | ----------------------------------- |
| `TaskLocal`          | Да                            | Да                          | Полная   | Основной выбор     | Контекст запроса, user ID, trace ID |
| Явные параметры      | Нет                           | Нет                         | Ручная   | Простые случаи     | Когда контекст небольшой            |
| `@globalActor`       | Да                            | Да (глобально)              | Полная   | Глобальные сервисы | Логи, аналитика, БД                 |
| Environment / DI     | Да (через @Environment)       | Частично                    | Ручная   | [[SwiftUI]]        | UI-контекст                         |
| Thread-local storage | Да                            | Нет                         | Нет      | Legacy             | Старый код                          |

**Вывод 2026**:
- `TaskLocal` — **самый чистый и современный** способ передавать контекст по цепочке асинхронных вызовов  
- Заменяет **почти все** ThreadLocal, RequestContext, CurrentUserHolder и т.д.  
- Особенно полезен в **backend**, **API-сервисах**, **логах**, **трассировке** и **аутентификации**

### 6. Лучшие практики 2026 года

- **Используй TaskLocal** для:
  - request ID / trace ID  
  - текущий пользователь / токен  
  - tenant ID / организация  
  - locale / timezone  
  - correlation ID / logging context  
- **Всегда** оборачивай в `withValue { ... }` — это единственный безопасный способ изменения  
- **Не храни** в TaskLocal **большие объекты** — только ID, строки, маленькие структуры  
- **Не забывай** — значение автоматически **очищается** при выходе из `withValue`  
- **Для UI** — чаще используй `@Environment` / `@EnvironmentObject`  
- **Swift 6 strict concurrency** — TaskLocal полностью поддерживается и безопасен  
- **Тестирование** — легко мокать через `withValue` в тестах  
- **Документируй** — пиши комментарий «TaskLocal — контекст запроса»

**Короткий девиз 2026**:
> «TaskLocal — это когда ты хочешь передать контекст по всей цепочке async-вызовов, **не засоряя сигнатуры функций** аргументами.  
> В 2026 году это **must-have** для любого серьёзного асинхронного приложения.»
