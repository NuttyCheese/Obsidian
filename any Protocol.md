**`any Protocol`** — это **экзистенциальный тип** ([[existential type]]) в [[Swift]], введённый в Swift 5.6–5.7 и ставший основным способом работы с протоколами начиная с **Swift 5.7+** (особенно после полного перехода на **strict concurrency** в Swift 6).

### Коротко и чётко: в чём суть

| Конструкция       | Что означает                                 | Диспетчеризация | Хранение в коллекции | Возвращаемый тип функции | Производительность | Когда использовать (2026)      |
| ----------------- | -------------------------------------------- | --------------- | -------------------- | ------------------------ | ------------------ | ------------------------------ |
| `any Protocol`    | «любой тип, который соответствует протоколу» | Динамическая    | Да                   | Можно                    | Медленнее          | Коллекции, свойства, хранилища |
| [[some Protocol]] | «конкретный, но неизвестный снаружи тип»     | Статическая     | Нет                  | Только возвращаемый      | Быстрее            | Возвращаемый тип функции       |

**Главное правило 2026 года**  
> [[any]] — это **коробка**, в которую можно положить **любой** conforming тип (но за это платим динамической диспетчеризацией).  
> [[some]] — это **конкретный** тип, который компилятор знает, но скрывает от вызывающего (статическая диспетчеризация, максимальная производительность).

### 1. Самые частые сценарии использования any Protocol

#### Сценарий 1 — Коллекция разных типов

```swift
protocol Drawable {
    func draw()
}

struct Circle: Drawable {
    func draw() { print("○") }
}

struct Square: Drawable {
    func draw() { print("■") }
}

// Правильно
let shapes: [any Drawable] = [Circle(), Square()]
shapes.forEach { $0.draw() }

// Ошибка компиляции (нельзя)
let shapesWrong: [Drawable] = ...     // Protocol 'Drawable' can only be used as a generic constraint
```

#### Сценарий 2 — Свойство, которое может хранить любой conforming тип

```swift
class Canvas {
    var currentShape: (any Drawable)?   // ✅ можно хранить любой Drawable
    
    func setShape(_ shape: any Drawable) {
        currentShape = shape
    }
}
```

#### Сценарий 3 — Функция, возвращающая any (редко, но возможно)

```swift
func randomShape() -> any Drawable {
    Bool.random() ? Circle() : Square()
}
```

### 2. Сравнение [[any]] vs [[some]] на реальных примерах

#### Возвращаемый тип функции — some (рекомендуется)

```swift
protocol ViewModel {
    var title: String { get }
}

struct UserViewModel: ViewModel {
    let title = "Профиль пользователя"
}

func makeUserViewModel() -> some ViewModel {
    UserViewModel()  // конкретный тип скрыт
}

// Компилятор знает точный тип → статическая диспетчеризация → быстрее
```

#### Коллекция или свойство — только any

```swift
let viewModels: [any ViewModel] = [
    UserViewModel(),
    ProductViewModel(),
    SettingsViewModel()
]

// any → динамическая диспетчеризация → чуть медленнее, но можно хранить разные типы
```

### 3. Таблица: any vs some — когда что выбирать (2026)

| Ситуация                                     | Использовать                        | Почему                                                       |
| -------------------------------------------- | ----------------------------------- | ------------------------------------------------------------ |
| Возвращаемый тип функции / computed property | `some Protocol`                     | Максимальная производительность, статическая диспетчеризация |
| Свойство класса / структуры                  | `any Protocol` (или конкретный тип) | Нужно хранить разные реализации                              |
| Коллекция / массив / словарь                 | `[any Protocol]`                    | Единственный способ хранить разные conforming типы           |
| Параметр функции                             | `any Protocol`                      | Позволяет передать любой conforming тип                      |
| [[Generic]] constraint                       | `<T: Protocol>`                     | Когда нужен конкретный тип внутри функции                    |
| [[Protocol]] с [[associatedtype]]            | `any Protocol` или `some`           | `some` только если возвращаешь, `any` — если хранишь         |

### 4. Реальные примеры из [[iOS]]-приложений 2026

#### Пример 1 — Коллекция ViewModel’ов в [[SwiftUI]]/[[UIKit]]

```swift
protocol ListItemViewModel {
    var title: String { get }
    var subtitle: String? { get }
}

struct PostViewModel: ListItemViewModel {
    let title: String
    let subtitle: String?
}

struct MessageViewModel: ListItemViewModel {
    let title: String
    let subtitle: String?
}

class FeedViewModel {
    var items: [any ListItemViewModel] = []
    
    func load() {
        items = [
            PostViewModel(title: "Новый пост", subtitle: "2 часа назад"),
            MessageViewModel(title: "Сообщение от Анны", subtitle: "Привет!")
        ]
    }
}
```

#### Пример 2 — Хранение делегатов (часто any)

```swift
protocol UpdateDelegate: AnyObject {
    func didUpdateData()
}

class DataManager {
    private var observers: [any UpdateDelegate] = []
    
    func addObserver(_ observer: any UpdateDelegate) {
        observers.append(observer)
    }
    
    func notify() {
        observers.forEach { $0.didUpdateData() }
    }
}
```

#### Пример 3 — some в [[SwiftUI]] (самый частый случай)

```swift
struct ProfileView: View {
    var body: some View {
        VStack {
            Text("Профиль")
            // ...
        }
    }
}

// some View — конкретный тип, скрытый от вызывающего
```

### 5. Таблица: производительность и ограничения

| Конструкция     | Диспетчеризация | Размер в памяти                     | Можно ли хранить в коллекции | Можно ли возвращать из функции | Производительность |
| --------------- | --------------- | ----------------------------------- | ---------------------------- | ------------------------------ | ------------------ |
| `some Protocol` | Статическая     | Как конкретный тип                  | Нет                          | Да                             | Максимальная       |
| `any Protocol`  | Динамическая    | Больше (экзистенциальный контейнер) | Да                           | Да                             | Ниже на 10–30%     |
| `<T: Protocol>` | Статическая     | Как T                               | Нет                          | Нет (только [[generic]])       | Максимальная       |

### 6. Типичные ошибки и ловушки 2026 года

- Использование `some` в коллекции или свойстве → ошибка компиляции
- Использование `any` в возвращаемом типе функции → потеря производительности
- Забыть : [[AnyObject]] в делегатах → `weak var delegate` не скомпилируется
- Смешивание `any` и `some` в одном месте → сложно отлаживать
- Переход на Swift 6 strict concurrency → много предупреждений с `any`

### 7. Лучшие практики 2026

- Возвращаемый тип функции / computed property → **всегда** `some Protocol`
- Коллекции, свойства, хранилища → **только** `any Protocol`
- Делегаты → [[AnyObject]] & [[Protocol]] + [[weak]]
- Generic функции → `<T: Protocol>`
- Для максимальной производительности → избегай `any` в горячих путях
- В SwiftUI — `some View`, `some ViewModel` — стандарт
- В UIKit — `any Delegate` для массивов делегатов, `some` для возвращаемых значений

**Короткий девиз 2026**:
> «`some` — для скорости и возвращаемых типов.  
> `any` — для хранения и коллекций.  
> Не путай — и код будет быстрее и безопаснее.»
