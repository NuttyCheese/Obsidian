**`@globalActor`** — это **атрибут типа**, который превращает обычный [[actor]] (или [[struct]]/[[class]] с `static let shared`) в **глобальный актёр** — единственный экземпляр, доступный всему приложению.

После применения атрибута:

- тип становится **глобальным диспетчером** (global actor)  
- его `static let shared` — это **единственный экземпляр**  
- любой код, помеченный `@ИмяГлобальногоАктора`, **гарантированно выполняется** в изоляции этого актёра  
- компилятор автоматически вставляет [[await]] и проверяет изоляцию

**Самый известный пример** — встроенный в язык **`@MainActor`**.

```swift
@globalActor
public struct MainActor {
    public static let shared = MainActorActor() // внутренний тип
}
```

### 2. Как объявить свой глобальный актёр (правильный синтаксис 2026)

```swift
// Вариант 1 — самый чистый и рекомендуемый
@globalActor
actor DatabaseActor {
    // здесь вся логика работы с базой
    func save(_ data: Data) { ... }
}

// Вариант 2 — если нужен struct (как @MainActor)
@globalActor
struct Database {
    actor ActorType { ... }
    static let shared = ActorType()
}

// Вариант 3 — class (редко, но возможно)
@globalActor
class LoggerActor {
    static let shared = LoggerActor()
    func log(_ message: String) { ... }
}
```

**Обязательные требования**:
- должен быть **один** `static let shared`  
- тип, на который указывает `shared`, должен быть **актёром** ([[actor]])  
- атрибут `@globalActor` применяется **только к номинальному типу** ([[struct]]/[[class]]/[[actor]]/[[enum]])

### 3. Как использовать @globalActor в коде

#### Вариант А — пометить отдельную функцию / свойство

```swift
@Database
func saveUser(_ user: User) async {
    // этот код гарантированно выполняется внутри DatabaseActor
    await DatabaseActor.shared.save(user.data)
}
```

#### Вариант Б — пометить весь класс / struct

```swift
@Database
class DataManager {
    private var cache: [String: Data] = [:]
    
    func store(_ data: Data, key: String) {
        cache[key] = data  // безопасно — изоляция DatabaseActor
    }
    
    func get(_ key: String) -> Data? {
        cache[key]
    }
}
```

Теперь **любой вызов** методов `DataManager` будет автоматически изолирован через `DatabaseActor.shared`.

#### Вариант В — смешивание с @MainActor

```swift
@Database
class DatabaseService {
    func fetchUsers() async -> [User] { ... }
}

@MainActor
class UserListViewModel: ObservableObject {
    @Published var users: [User] = []
    
    func load() {
        Task {
            let fetched = await DatabaseService().fetchUsers()
            users = fetched  // безопасно — @MainActor
        }
    }
}
```

### 4. Типичные сценарии применения в 2026 году

| Сценарий                                        | Почему именно @globalActor                             | Альтернатива без глобального актёра |
| ----------------------------------------------- | ------------------------------------------------------ | ----------------------------------- |
| Доступ к [[Core Data]] / [[Swift/Теория/Сторонние библиотеки и расширения/Realm]] / [[SQLite]] | Один глобальный контекст — нет гонок                   | Передавать экземпляр actor-а везде  |
| Логирование / аналитика                         | Централизованное безопасное логирование                | actor + инъекция зависимостей       |
| Работа с [[Swift/Теория/Хранение данных/UserDefaults]] / [[KeyChain]]        | Последовательный доступ без лишних await               | Сериальная DispatchQueue            |
| Глобальный кэш / [[Singleton]]-ресурсы          | Единая точка синхронизации                             | @MainActor или обычный actor        |
| Миграция legacy-кода                            | Легко заменить [[DispatchQueue]].[[main]] / [[serial]] | Постепенно на обычные actor-ы       |

### 5. Сравнение @globalActor с другими подходами (2026)

| Подход                           | Глобальность | Изоляция по умолчанию | Отмена задач   | Сложность | Рекомендация 2026                      |
| -------------------------------- | ------------ | --------------------- | -------------- | --------- | -------------------------------------- |
| @globalActor                     | Да           | Полная                | Через Task     | Средняя   | Когда нужен глобальный singleton-актёр |
| Обычный actor + shared экземпляр | Нет          | Полная                | Через Task     | Низкая    | Почти всегда лучше                     |
| @MainActor                       | Да           | Полная (main thread)  | Через [[Task]] | Низкая    | Для всего UI                           |
| Сериальная DispatchQueue         | Нет          | Ручная                | Сложно         | Средняя   | Legacy-код                             |
| actor + инъекция зависимостей    | Нет          | Полная                | Легко          | Низкая    | Самый современный и рекомендуемый      |

**Вывод 2026**:
- **@globalActor** полезен, когда вам нужен **по-настоящему глобальный** контекст (как `@MainActor` для UI)  
- В 90 % случаев **лучше обычный actor + инъекция** (dependency injection)  
- Глобальные актёры усложняют тестирование и могут привести к **bottleneck** (узкому месту)

### 6. Лучшие практики 2026 года

- **Используйте @globalActor редко** — только когда нужен именно глобальный singleton-контекст  
- **Предпочитайте обычный actor** + передачу через init / Environment / DI  
- **Не злоупотребляйте** — каждый глобальный актёр = потенциальное узкое место  
- **Для UI** — используйте встроенный `@MainActor`  
- **Для тестов** — заменяйте глобальные актёры моками (например, `actor TestDatabaseActor: DatabaseActorProtocol`)  
- **Swift 6 strict concurrency** — глобальные актёры полностью поддерживаются и безопасны  
- **Документируйте** — пишите комментарий «global actor for centralized DB access»

**Короткий девиз 2026**:
> «@globalActor — это когда вам нужен свой личный @MainActor для чего-то кроме UI.  
> В 2026 году это мощный, но опасный инструмент.  
> В большинстве случаев лучше обычный actor + инъекция зависимостей.»
