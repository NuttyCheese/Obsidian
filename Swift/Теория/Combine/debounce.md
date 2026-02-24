**`debounce`** — один из самых полезных и часто используемых операторов в **[[Combine]]** (и в [[RxSwift]]/RxCocoa тоже).

Он **откладывает выдачу значения** на заданный интервал времени и пропускает его **только если** за это время не пришло новое значение.  
Если новое значение пришло раньше — таймер сбрасывается, и отсчёт начинается заново.

Проще говоря:  
`debounce` — это «подожди, пока пользователь перестанет печатать/скроллить/кликать, и только тогда обработай последнее значение».

### Зачем нужен debounce (самые частые сценарии 2025–2026)

| Сценарий                                   | Без debounce                      | С debounce                           | Почему это критично                         |
| ------------------------------------------ | --------------------------------- | ------------------------------------ | ------------------------------------------- |
| Поиск по мере ввода текста                 | Запрос на сервер на каждый символ | Запрос только после паузы 300–500 мс | Экономия трафика, меньше нагрузки на сервер |
| Валидация формы в реальном времени         | Проверка на каждый keystroke      | Проверка после паузы                 | Не раздражает пользователя миганием ошибок  |
| Автосохранение заметок / черновиков        | Сохранение на каждый символ       | Сохранение после паузы 1–2 секунды   | Меньше операций ввода-вывода                |
| Обработка быстрых событий (скролл, свайпы) | Обновление UI 60 раз в секунду    | Обновление только после остановки    | Плавность и экономия [[CPU]]/[[GPU]]        |
| Отправка аналитики при взаимодействии      | 10 событий "scroll" подряд        | Только одно событие после паузы      | Чистая аналитика без шума                   |

### Самые популярные и рекомендуемые паттерны (2026)

#### 1. Поиск по мере ввода (самый частый кейс)

```swift
class SearchViewModel: ObservableObject {
    
    @Published var searchText = ""
    @Published var results: [String] = []
    
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        $searchText
            .debounce(for: .milliseconds(400), scheduler: DispatchQueue.main)
            .removeDuplicates()                     // не искать одинаковые запросы
            .filter { !$0.isEmpty }                 // игнорируем пустой запрос
            .flatMap { [weak self] query in
                self?.search(query: query) ?? Empty(completeImmediately: true).eraseToAnyPublisher()
            }
            .receive(on: DispatchQueue.main)
            .sink { [weak self] newResults in
                self?.results = newResults
            }
            .store(in: &cancellables)
    }
    
    private func search(query: String) -> AnyPublisher<[String], Never> {
        // Здесь может быть реальный сетевой запрос
        Just(["Результат 1 для \(query)", "Результат 2 для \(query)"])
            .eraseToAnyPublisher()
    }
}
```

#### 2. Валидация формы с отложенной проверкой

```swift
class FormViewModel: ObservableObject {
    
    @Published var email = ""
    @Published var isEmailValid = false
    
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        $email
            .debounce(for: .milliseconds(600), scheduler: DispatchQueue.main)
            .map { email in
                email.contains("@") && email.contains(".")
            }
            .assign(to: &$isEmailValid)
    }
}
```

В контроллере:

```swift
viewModel.$isEmailValid
    .assign(to: \.isHidden, on: errorLabel)
    .store(in: &cancellables)
```

#### 3. Автосохранение черновика (очень популярный паттерн)

```swift
class NoteViewModel: ObservableObject {
    
    @Published var text = ""
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        $text
            .debounce(for: .seconds(2), scheduler: DispatchQueue.global(qos: .background))
            .sink { [weak self] newText in
                self?.saveToDisk(text: newText)
            }
            .store(in: &cancellables)
    }
    
    private func saveToDisk(text: String) {
        // сохранение в файл / Core Data / Realm
        print("Сохранено:", text)
    }
}
```

### Полезные комбинации с debounce (2026 топ)

| Комбинация                              | Когда использовать                                      | Пример |
|-----------------------------------------|----------------------------------------------------------|--------|
| `debounce` + `removeDuplicates`         | Поиск, чтобы не дублировать одинаковые запросы           | Почти всегда перед сетью |
| `debounce` + `filter { !$0.isEmpty }`   | Игнорировать пустой ввод                                 | Поисковые строки |
| `debounce` + `flatMap`                  | Отложенный сетевой запрос                                | Поиск по API |
| `debounce` + `throttle`                 | Когда нужен максимум 1 событие в N секунд (редко)        | Быстрые клики |
| `debounce` + `receive(on: .main)`       | Обновление UI после паузы                                | Всегда для UI |

### Лучшие практики debounce в Combine 2026

- **Интервал**:
  - Поиск по тексту → 300–500 мс  
  - Автосохранение → 1–3 секунды  
  - Валидация → 600–800 мс  
- **Scheduler**:
  - UI-обновления → `RunLoop.main` или `DispatchQueue.main`  
  - Фоновые операции → `DispatchQueue.global(qos: .userInitiated)`  
- **Всегда** добавляй `.removeDuplicates()` перед debounce — экономит запросы  
- **Не забывай** `.store(in: &cancellables)` — иначе подписка будет жить вечно  
- **Для SwiftUI** — используй `.debounce` в `.onChange` или в ViewModel  
- **Документируй** — пиши комментарий:

```swift
// Отложенный поиск: ждём 400 мс после последнего ввода
$searchText
    .debounce(for: .milliseconds(400), scheduler: RunLoop.main)
    .removeDuplicates()
    .flatMap { self.search(query: $0) }
    .assign(to: &$results)
```

**Короткий итог 2026**:
> `debounce` — оператор, который **ждёт паузу** в потоке и выдаёт **только последнее значение** после неё.  
> В 2026 году:  
> - самый популярный кейс — поиск по мере ввода, автосохранение, отложенная валидация  
> - интервал 300–800 мс — золотая середина  
> - комбинируй с `removeDuplicates()`, `filter`, `flatMap`, `receive(on: .main)`  
> - это **must-have** оператор для любого реактивного UI в Combine  

Удачи с отзывчивым и ненагружающим сервер поиском и формами! 🔍