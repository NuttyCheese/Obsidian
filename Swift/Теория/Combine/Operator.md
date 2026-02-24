**Operators** в **Combine** — это методы, которые **преобразуют**, **фильтруют**, **комбинируют** или **управляют** потоками данных от одного или нескольких `Publisher`.  

Важно понимать:  
операторы **не выполняют** код сразу — они создают **новый Publisher**, который применяет трансформацию к исходному потоку.  
Чтобы поток «ожил» и начал выдавать значения — нужна **подписка** (`sink`, `assign`, `store(in: &cancellables)` и т.д.).

### Основные категории операторов (актуально на 2026 год)

| Категория                              | Что делают операторы                                       | Самые популярные операторы (топ-использование 2026) |
|----------------------------------------|------------------------------------------------------------|-------------------------------------------------------|
| **Трансформация значений**             | Изменяют каждое значение                                   | `map`, `compactMap`, `tryMap`, `replaceNil(with:)`    |
| **Фильтрация**                         | Пропускают только нужные значения                          | `filter`, `removeDuplicates`, `dropFirst`, `prefix`   |
| **Комбинирование нескольких потоков**  | Объединяют данные из 2+ Publisher                          | `combineLatest`, `merge`, `zip`, `withLatestFrom`-подобные |
| **Обработка ошибок и завершений**      | Ловят ошибки, заменяют их, повторяют попытки              | `catch`, `replaceError`, `retry`, `retry(attempts:)`  |
| **Управление временем**                | Задержка, таймауты, группировка по времени                 | `debounce`, `throttle`, `delay`, `timeout`            |
| **Планирование и потоки**              | Переключают выполнение на нужный поток                     | `receive(on:)`, `subscribe(on:)`                      |
| **Преобразование типа Publisher**      | Скрывают конкретный тип, делают общий                      | `eraseToAnyPublisher()`, `setFailureType(to:)`        |

### Топ-15 самых используемых операторов в реальных проектах (2026)

| Оператор                  | Что делает простыми словами                                 | Самый частый кейс в 2026                              | Пример |
|---------------------------|-------------------------------------------------------------|-------------------------------------------------------|--------|
| `map`                     | Преобразует каждое значение                                 | Изменение типа, форматирование                        | `.map { $0.uppercased() }` |
| `compactMap`              | `map` + фильтр `nil`                                        | Извлечение optional, парсинг                         | `.compactMap { Int($0) }` |
| `filter`                  | Пропускает только подходящие значения                       | Отсечение пустых строк, invalid данных               | `.filter { $0.count > 3 }` |
| `debounce`                | Ждёт паузу, выдаёт последнее значение                       | Поиск по мере ввода, автосохранение                  | `.debounce(for: .seconds(0.5), scheduler: .main)` |
| `removeDuplicates`        | Убирает повторяющиеся подряд значения                       | Избегаем лишних запросов при одинаковом вводе         | `.removeDuplicates()` |
| `combineLatest`           | Последние значения всех потоков при изменении любого        | Валидация формы (email + пароль + согласие)           | `CombineLatest3($email, $password, $agreed)` |
| `receive(on:)`            | Переключает выполнение на указанный поток                   | Обновление UI на главном потоке                       | `.receive(on: DispatchQueue.main)` |
| `assign(to:)`             | Присваивает значение `@Published` свойству                  | Прямая привязка Publisher → UI                        | `.assign(to: &$isValid)` |
| `sink`                    | Основной способ подписки (completion + value)               | Всё, что не `assign`                                  | `.sink { print($0) }` |
| `catch`                   | Ловит ошибку и заменяет на новый Publisher                  | Показ fallback UI при ошибке сети                     | `.catch { Just([]) }` |
| `retry`                   | Повторяет подписку при ошибке (до N раз)                    | Сетевые запросы с повтором                            | `.retry(3)` |
| `eraseToAnyPublisher`     | Скрывает конкретный тип Publisher                           | Возврат из функции                                    | `.eraseToAnyPublisher()` |
| `flatMap`                 | Преобразует каждое значение в новый Publisher               | Сетевые запросы после ввода                           | `.flatMap { self.fetch($0) }` |
| `zip`                     | Ждёт значение от каждого Publisher и выдаёт кортеж         | Параллельные запросы (один раз)                       | `zip(fetchA(), fetchB())` |
| `merge`                   | Объединяет значения из нескольких Publisher по мере поступления | Объединение событий из разных источников              | `.merge(with: otherPublisher)` |

### Полный реальный пример (поиск + валидация + загрузка)

```swift
class SearchViewModel: ObservableObject {
    
    @Published var query = ""
    @Published var results: [String] = []
    @Published var isLoading = false
    
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        $query
            .debounce(for: .milliseconds(400), scheduler: RunLoop.main)      // ждём паузу в вводе
            .removeDuplicates()                                             // не ищем одинаковое
            .filter { !$0.isEmpty }                                         // игнорируем пустой запрос
            .handleEvents(receiveOutput: { [weak self] _ in
                self?.isLoading = true
            })
            .flatMap { [weak self] text in
                self?.search(query: text) ?? Empty(completeImmediately: true).eraseToAnyPublisher()
            }
            .catch { _ in Just([]) }                                        // если ошибка — пустой массив
            .receive(on: DispatchQueue.main)
            .sink { [weak self] newResults in
                self?.results = newResults
                self?.isLoading = false
            }
            .store(in: &cancellables)
    }
    
    private func search(query: String) -> AnyPublisher<[String], Error> {
        // имитация сети
        Future { promise in
            DispatchQueue.global().asyncAfter(deadline: .now() + 0.8) {
                promise(.success(["Результат 1 для \(query)", "Результат 2"]))
            }
        }
        .eraseToAnyPublisher()
    }
}
```

### Лучшие практики Combine operators 2026

- **Всегда** добавляй `.receive(on: DispatchQueue.main)` перед `.assign` / `.sink` для UI-обновлений  
- **Используй** `debounce` + `removeDuplicates` перед любым сетевым запросом по вводу  
- **Для ошибок** — `.catch`, `.replaceError`, `.retry` — делай пайплайн устойчивым  
- **Для форм** — `combineLatest` + `map` — золотой стандарт валидации  
- **Для тестов** — `Just`, `Fail`, `Deferred`, `PassthroughSubject` — твои лучшие друзья  
- **Документируй** — пиши комментарий:

```swift
// Поиск с debounce 400 мс + фильтрация пустых запросов
$searchText
    .debounce(for: .milliseconds(400), scheduler: RunLoop.main)
    .removeDuplicates()
    .filter { !$0.isEmpty }
    .flatMap { self.search(query: $0) }
    .assign(to: &$results)
```

**Короткий итог 2026**:
> Operators — это **трансформеры потоков** в Combine: они создают новый Publisher с изменённым поведением.  
> Самые важные в 2026:  
> - `map`, `compactMap`, `filter`, `debounce`, `combineLatest`, `receive(on:)`, `assign`, `sink`  
> - используй их для: поиска, валидации форм, сетевых цепочек, обработки ошибок  
> - всегда храни подписку в `Set<AnyCancellable>`  
> - это **основа** любого реактивного кода в Combine  

Удачи с мощными и читаемыми потоками данных в твоём проекте! 🔄