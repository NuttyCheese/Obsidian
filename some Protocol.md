**`some Protocol`** — это **opaque type** (непрозрачный тип) в [[Swift]], введённый в Swift 5.1 и ставший одним из самых важных инструментов языка после Swift 5.7 и особенно в Swift 6 с полным **strict concurrency checking**.

### Коротко и по делу

| Конструкция      | Что это значит на человеческом языке                         | Диспетчеризация | Можно ли хранить в массиве/словаре | Можно ли возвращать из функции | Производительность | Главное ограничение        |
| ---------------- | ------------------------------------------------------------ | --------------- | ---------------------------------- | ------------------------------ | ------------------ | -------------------------- |
| `some Protocol`  | «Я точно знаю конкретный тип, но не скажу тебе какой именно» | Статическая     | Нет                                | Да (только возвращаемый тип)   | Максимальная       | Нельзя хранить в коллекции |
| [[any Protocol]] | «Любой тип, который соответствует протоколу»                 | Динамическая    | Да                                 | Да                             | Ниже на 10–30%     | Можно хранить где угодно   |

### Когда использовать `some` (самые частые случаи 2026 года)

1. **Возвращаемый тип функции** (самое популярное место)

```swift
protocol ViewModel {
    var title: String { get }
    func loadData() async
}

func makeUserViewModel() -> some ViewModel {
    UserViewModel()  // конкретный тип скрыт от вызывающего
}

let vm = makeUserViewModel()  // компилятор знает точный тип → статическая диспетчеризация
```

2. **Computed property в [[SwiftUI]] / [[UIKit]]**

```swift
struct ProfileScreen: View {
    var body: some View {
        VStack {
            Text("Профиль")
            // ...
        }
    }
}
```

3. **Метод в протоколе или классе**

```swift
protocol Factory {
    func createService() -> some Service
}

struct RealFactory: Factory {
    func createService() -> some Service {
        NetworkService()
    }
}
```

### Когда `some` НЕЛЬЗЯ использовать

- В свойствах структуры/класса (кроме computed property с getter)
- В массивах / словарях / коллекциях
- В параметрах функций (только `any`)
- В переменных, которые нужно присваивать разные conforming типы

```swift
// Ошибка компиляции
var viewModels: [some ViewModel] = []           // ❌

// Правильно
var viewModels: [any ViewModel] = []            // ✅
```

### Сравнение производительности и размера (2026 данные)

| Конструкция     | Диспетчеризация | Размер в памяти (примерно)                 | Скорость вызова метода | Пример использования в горячих путях |
| --------------- | --------------- | ------------------------------------------ | ---------------------- | ------------------------------------ |
| `some Protocol` | Статическая     | Как конкретный тип (~16–32 байта)          | Максимальная           | SwiftUI body, ViewModel-фабрики      |
| `any Protocol`  | Динамическая    | + 16–32 байта (экзистенциальный контейнер) | Ниже                   | Коллекции, делегаты, хранилища       |
| `<T: Protocol>` | Статическая     | Как T                                      | Максимальная           | [[Generic]]-функции и методы         |

### Реальные примеры из iOS-приложений 2026

#### Пример 1 — SwiftUI View (самый частый случай)

```swift
struct ContentView: View {
    var body: some View {
        NavigationStack {
            List {
                Text("Главная")
            }
            .navigationTitle("Приложение")
        }
    }
}
```

#### Пример 2 — Фабрика ViewModel

```swift
protocol ViewModel: ObservableObject {
    var title: String { get }
}

struct UserViewModel: ViewModel {
    @Published var title = "Профиль"
}

func makeViewModel(for user: User) -> some ViewModel {
    if user.isPremium {
        PremiumUserViewModel(user: user)
    } else {
        UserViewModel(user: user)
    }
}
```

#### Пример 3 — Ошибка, которую часто делают новички в Swift 6

```swift
// ❌ Ошибка в Swift 6 strict concurrency
class DataManager {
    var items: [any Identifiable] = []   // any — динамическая диспетчеризация
}

// ✅ Лучше (если возможно)
class DataManager<T: Identifiable> {
    var items: [T] = []                  // конкретный тип → статическая диспетчеризация
}
```

### Таблица: когда выбирать any vs some (2026 золотой стандарт)

| Место в коде                             | Рекомендация    | Почему                                               |
| ---------------------------------------- | --------------- | ---------------------------------------------------- |
| `var body: some View` (SwiftUI)          | `some`          | Максимальная производительность SwiftUI              |
| `func makeViewModel() -> some ViewModel` | [[some]]        | Скрываем конкретный тип, статическая диспетчеризация |
| `let delegates: [any Delegate]`          | [[any]]         | Нужно хранить разные реализации                      |
| `weak var delegate: (any Delegate)?`     | `any`           | Свойство хранит значение                             |
| `func process<T: Equatable>(_ value: T)` | `<T: Protocol>` | Generic-функция — максимальная скорость              |
| `let cache: [String: any Codable]`       | `any`           | Разные типы в словаре                                |

### Ключевые правила 2026 года (Swift 6 strict concurrency)

1. **Возвращаемые типы** — **всегда** `some Protocol` (кроме случаев, когда нужен any)
2. **Коллекции и свойства** — **только** `any Protocol`
3. **Делегаты** — [[AnyObject]] & [[Protocol]] + [[weak]]
4. **Generic-функции и методы** — `<T: Protocol>`
5. **SwiftUI** — `some View`, `some ViewModel` — золотой стандарт
6. **UIKit** — `any Delegate` для массивов, `some` для фабрик
7. **Производительность** — в горячих путях избегай `any` (особенно в циклах)

**Короткий девиз 2026**:
> `some` — для скорости и возвращаемых типов.  
> `any` — для хранения и коллекций.  
> Не путай — и твой код будет быстрее, безопаснее и современнее.
