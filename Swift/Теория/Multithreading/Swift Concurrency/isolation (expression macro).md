Вот **полное, подробное и максимально актуальное** (на февраль 2026 года) руководство по макросу **`#isolation`** в Swift — как он работает, зачем нужен и как правильно его использовать.

### 1. Что такое #isolation (простыми словами)

**`#isolation`** — это **compile-time expression macro** (макрос-выражение), который возвращает **тип изоляции** (isolation type) текущего выражения или контекста.

Он отвечает на вопрос:

> «В каком изолированном контексте (actor, MainActor, unsafe/global executor и т.д.) выполняется этот код прямо сейчас?»

Макрос **не влияет** на runtime — он работает **только на этапе компиляции** и используется для:

- анализа кода  
- генерации предупреждений / ошибок  
- условной компиляции  
- создания безопасных библиотек и инструментов  
- отладки изоляции

**Коротко**:
> `#isolation` = «компилятор, скажи мне, в каком executor-е / actor-е я сейчас нахожусь».

### 2. Возвращаемые значения #isolation (2026)

Макрос возвращает один из следующих вариантов (тип — это **метатип**, не runtime-значение):

| Значение #isolation       | Описание                                                                 | Когда возвращается |
|----------------------------|--------------------------------------------------------------------------|---------------------|
| `.MainActor`               | Код выполняется в контексте `@MainActor` (главный поток)                 | Внутри `@MainActor` функции / класса / замыкания |
| `ActorType` (конкретный actor) | Код выполняется внутри конкретного `actor` (например, `MyDatabaseActor`) | Внутри метода / свойства обычного `actor` |
| `.unsafe`                  | Нет изоляции — глобальный executor или detached task                     | В `Task { ... }`, `Task.detached`, global executor |
| `.unknown`                 | Компилятор не может точно определить изоляцию                            | Сложные случаи, generic код, некоторые макросы |
| `isolated(any) ActorType`  | Параметр помечен `isolated(any)` — изоляция наследуется от аргумента     | Внутри функции с `isolated(any)` параметром |

### 3. Самые популярные шаблоны использования #isolation в 2026

#### Шаблон 1 — Проверка MainActor (самый частый в UI-библиотеках)

```swift
@MainActor
func updateUI() {
    let iso = #isolation(self)
    if iso == MainActor.self {
        print("Мы на главном потоке — безопасно обновляем UI")
    } else {
        fatalError("updateUI вызван не с MainActor")
    }
    
    // или switch
    switch #isolation(self) {
    case MainActor.self:
        label.text = "Обновлено"
    default:
        fatalError("Неверный контекст")
    }
}
```

#### Шаблон 2 — Проверка внутри actor-а

```swift
actor Database {
    var users: [User] = []
    
    func addUser(_ user: User) {
        let iso = #isolation(self)
        print("Текущая изоляция:", iso)  // → Database
        
        users.append(user)  // безопасно
    }
}
```

#### Шаблон 3 — Проверка isolated параметра

```swift
actor Counter {
    var value = 0
}

func increment(_ counter: isolated(any) Counter, by amount: Int) {
    let iso = #isolation(counter)
    print("Изоляция параметра:", iso)  // → Counter
    
    counter.value += amount  // без await — потому что isolated
}
```

#### Шаблон 4 — Отладка detached / unsafe контекста

```swift
Task.detached {
    let iso = #isolation(self)
    print("Изоляция в detached задаче:", iso)  // → unsafe
    
    if iso == .unsafe {
        print("Нет изоляции — будь осторожен с mutable данными")
    }
}
```

#### Шаблон 5 — Условная компиляция / предупреждения в библиотеке

```swift
func safeLog(_ message: String) {
    switch #isolation(self) {
    case MainActor.self:
        print("[UI] \(message)")
    case is MyDatabaseActor.Type:
        print("[DB] \(message)")
    case .unsafe:
        #warning("Логирование вызвано без изоляции — возможна гонка")
        print("[UNSAFE] \(message)")
    default:
        print("[?]", message)
    }
}
```

### 4. Типичные ошибки и ловушки 2026 года

| Ошибка                                      | Последствия                              | Как избежать |
|---------------------------------------------|------------------------------------------|--------------|
| Использовать #isolation в runtime-коде      | Нет смысла — это compile-time            | Только для compile-time проверок / генерации |
| Думать, что #isolation возвращает runtime-значение | Нет — это метатип                        | Использовать только в switch / if с типами |
| Проверять #isolation(self) вне actor / MainActor | Вернёт `.unsafe` или `.unknown`          | Проверять только в нужном контексте |
| Забыть, что #isolation работает на этапе компиляции | Нельзя использовать в if / switch runtime | Только compile-time логика |
| Ожидать точного имени актёра                | Возвращает тип, а не строку              | Использовать `is MyActor.Type` или `== MyActor.self` |

### 5. #isolation vs другие механизмы анализа контекста (2026)

| Механизм                  | Когда работает       | Runtime / Compile-time | Может бросить ошибку | Рекомендация 2026 |
|---------------------------|----------------------|------------------------|-----------------------|-------------------|
| `#isolation`              | Compile-time         | Compile-time           | Нет                   | Основной для анализа |
| `Task.isCurrentActor`     | Runtime              | Runtime                | Нет                   | Редко, для отладки |
| `#if swift(>=6.0)`        | Compile-time         | Compile-time           | Нет                   | Условная компиляция |
| `Thread.isMainThread`     | Runtime              | Runtime                | Нет                   | Legacy / отладка |
| `actor` / `@MainActor`    | Compile-time + runtime | Оба                  | Да (ошибка компиляции) | Основная защита   |

**Вывод 2026**:
- `#isolation` — **самый мощный compile-time инструмент** для анализа изоляции  
- Используется **в библиотеках, макросах, инструментах анализа кода**  
- Не предназначен для runtime-логики (для этого — `Task.isCurrentActor`, `Thread.isMainThread` и т.д.)

### 6. Лучшие практики 2026 года

- **Используй #isolation** в:
  - библиотеках / фреймворках  
  - собственных макросах  
  - инструментах анализа кода  
  - местах, где нужно генерировать предупреждения / ошибки compile-time
- **Не используй** в обычном application-коде — это избыточно  
- **Документируй** — пиши комментарий «compile-time isolation check»  
- **Swift 6 strict concurrency** — `#isolation` полностью поддерживается и очень полезен  
- **Тестирование** — проверяй, что макросы / предупреждения работают в разных контекстах

**Короткий девиз 2026**:
> «#isolation — это когда ты спрашиваешь у компилятора: «в каком executor-е / actor-е я сейчас?».  
> Это инструмент для библиотек и макросов, а не для обычного кода приложения.»

Удачи с глубоким анализом изоляции и безопасным compile-time кодом в Swift! 🔍