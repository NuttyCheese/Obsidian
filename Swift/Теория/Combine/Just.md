**`Just`** — это самый простой и часто используемый **Publisher** в **Combine**, который:

- выпускает **ровно одно значение** (Output)
- сразу после этого **завершается** (finished)
- **никогда не выдаёт ошибку** (Failure = Never)

Это идеальный инструмент для:
- создания Publisher из константы
- возврата фиксированного значения в цепочке операторов
- тестирования и мокинга
- быстрого старта пайплайна
- интеграции синхронных данных в реактивный поток

### Основные характеристики Just (актуально 2025–2026)

| Характеристика                  | Значение / Поведение                              | Важно помнить |
|---------------------------------|----------------------------------------------------|---------------|
| Количество значений             | **Ровно одно**                                     | После него — finished |
| Ошибки                          | **Never** (не может завершиться с ошибкой)         | Для ошибок используй `Fail` |
| Завершение                      | Автоматически после первого значения               | `.finished` сразу после value |
| Тип                             | `Just<Output>` где Failure = `Never`               | Очень предсказуемый |
| Когда завершается               | Сразу после `.send(value)` в sink                  | Нет задержки |
| Эквивалент в RxSwift            | `Observable.just(value)`                           | — |

### Самые популярные и полезные паттерны использования Just (2026)

#### 1. Простейший Just (самый частый)

```swift
import Combine

Just("Привет, Combine!")
    .sink { value in
        print("Получено:", value)          // → "Получено: Привет, Combine!"
    }
    .store(in: &cancellables)
```

#### 2. Just как замена опционала или дефолтного значения

```swift
@Published var selectedUser: User?

selectedUserPublisher
    .map { $0 ?? User.guest }              // если nil → гостевой пользователь
    .map { $0.name }
    .assign(to: &$displayName)
```

Или короче:

```swift
$selectedUser
    .map { $0?.name ?? "Гость" }
    .assign(to: &$displayName)
```

#### 3. Just в цепочке операторов (очень популярно)

```swift
$searchText
    .debounce(for: .milliseconds(400), scheduler: RunLoop.main)
    .removeDuplicates()
    .flatMap { query in
        if query.isEmpty {
            return Just([]).eraseToAnyPublisher()           // ← пустой результат
        } else {
            return searchService.search(query: query)
        }
    }
    .assign(to: &$searchResults)
```

#### 4. Just для тестирования и мокинга

```swift
func testUserFetch() {
    let mockPublisher = Just(User.mock)
        .setFailureType(to: NetworkError.self)
        .eraseToAnyPublisher()
    
    // подмена реального сервиса на mock в тестах
}
```

#### 5. Just + delay (имитация задержки)

```swift
Just("Загрузка завершена")
    .delay(for: .seconds(1.5), scheduler: RunLoop.main)
    .sink { print($0) }
    .store(in: &cancellables)
```

#### 6. Just в flatMap / switchMap (управление состоянием загрузки)

```swift
isLoading = true

fetchData()
    .map { Result.success($0) }
    .catch { Just(Result.failure($0)) }
    .prepend(Result.loading)                     // ← сразу покажем загрузку
    .receive(on: DispatchQueue.main)
    .assign(to: &$resultState)
```

### Сравнение Just с другими Publisher (2026)

| Publisher          | Сколько значений | Завершается сразу? | Может иметь ошибку? | Когда использовать |
|--------------------|-------------------|---------------------|----------------------|--------------------|
| `Just(value)`      | 1                 | Да                  | Нет (Never)          | фиксированное значение, мок, дефолт |
| `Fail(error)`      | 0                 | Да (сразу ошибка)   | Да                   | явная ошибка в цепочке |
| `Empty(completeImmediately: true)` | 0             | Да                  | Нет                  | пустой успешный результат |
| `Empty(completeImmediately: false)` | 0            | Нет                 | Нет                  | бесконечный пустой поток |
| `Deferred { Just(value) }` | 1             | Да                  | Нет                  | отложенное создание |
| `CurrentValueSubject` | 1 + изменения    | Нет                 | Нет                  | текущее значение + обновления |
| `PassthroughSubject` | 0 + изменения     | Нет                 | Нет                  | события без начального значения |

### Лучшие практики Just в Combine 2026

- **Используйте** `Just` вместо `return Just(value).eraseToAnyPublisher()` — это стандартный способ возвращать фиксированный результат из функции  
- **Комбинируйте** с `.setFailureType(to:)` если нужно добавить тип ошибки в цепочку  
- **Не используйте** `Just` для повторяющихся событий — для этого `CurrentValueSubject` / `@Published`  
- **Для ошибок** — используйте `Fail(error:)` или `Future`  
- **В тестах** — `Just` + `XCTestExpectation` — классика  
- **Документируйте** — пишите комментарий:

```swift
/// Возвращает фиксированный результат без сетевого запроса (для тестов или fallback)
func mockFetch() -> AnyPublisher<[User], Never> {
    Just([User.mock]).eraseToAnyPublisher()
}
```

**Короткий итог 2026**:
> `Just` — это **Publisher**, который **один раз** выдаёт значение и сразу завершается.  
> В 2026 году:  
> - самый частый кейс — возврат фиксированного значения, мокинг, дефолт в цепочке  
> - идеален для `flatMap`, `catch`, `prepend`, `replaceNil`  
> - не подходит для повторяющихся событий (для этого `@Published`, Subject)  
> - это **самый простой** и **самый надёжный** способ вставить константу в Combine-пайплайн  

Удачи с чистыми и предсказуемыми потоками данных в твоём проекте! 📡