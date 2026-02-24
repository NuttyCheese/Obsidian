**`tryMap`** — это оператор в **[[Combine]]**, который работает почти как обычный `map`, но **может выбросить ошибку** внутри замыкания.

Если внутри `tryMap` возникает исключение ([[throw]]), поток **немедленно завершается с ошибкой** (`Failure`), и дальнейшие операторы не выполняются.

Это делает `tryMap` идеальным инструментом для операций, где преобразование может завершиться неудачей (парсинг, декодирование, деление на ноль и т.д.).

### Ключевые отличия tryMap от map

| Оператор | Может бросить ошибку? | Что происходит при ошибке?                | Когда использовать в 2026                                              |
| -------- | --------------------- | ----------------------------------------- | ---------------------------------------------------------------------- |
| [[map]]  | Нет                   | Ошибка внутри замыкания → краш приложения | Простые безопасные преобразования                                      |
| `tryMap` | Да                    | Поток завершается с `.failure(error)`     | Любое преобразование, которое может fail (парсинг, decode, вычисления) |

### Самые популярные и рекомендуемые сценарии использования tryMap (2026)

#### 1. Парсинг строк в числа / даты / [[enum]]

```swift
["10", "20", "тридцать", "40"].publisher
    .tryMap { str -> Int in
        guard let number = Int(str) else {
            throw NSError(domain: "ParseError", code: 1, userInfo: [NSLocalizedDescriptionKey: "Не число: \(str)"])
        }
        return number * 2
    }
    .sink(
        receiveCompletion: { completion in
            switch completion {
            case .finished: print("Успешно завершено")
            case .failure(let error): print("Ошибка парсинга:", error.localizedDescription)
            }
        },
        receiveValue: { doubled in
            print("Удвоенное число:", doubled)
        }
    )
    .store(in: &cancellables)
```

Вывод:
```
Удвоенное число: 20
Удвоенное число: 40
Удвоенное число: 80
Ошибка парсинга: Не число: тридцать
```

#### 2. Декодирование [[JSON]] (самый частый кейс в реальных приложениях)

```swift
URLSession.shared.dataTaskPublisher(for: url)
    .map(\.data)
    .tryMap { data -> UserProfile in
        let decoder = JSONDecoder()
        decoder.keyDecodingStrategy = .convertFromSnakeCase
        return try decoder.decode(UserProfile.self, from: data)
    }
    .receive(on: DispatchQueue.main)
    .sink(
        receiveCompletion: { completion in
            if case .failure(let error) = completion {
                print("Ошибка декодирования:", error)
                // показать алерт
            }
        },
        receiveValue: { [weak self] profile in
            self?.updateUI(with: profile)
        }
    )
    .store(in: &cancellables)
```

#### 3. Безопасное деление / математические операции

```swift
[10, 5, 0, 2].publisher
    .tryMap { number -> Double in
        guard number != 0 else {
            throw DivisionByZeroError()
        }
        return 100.0 / Double(number)
    }
    .sink(
        receiveCompletion: { print("Completion:", $0) },
        receiveValue: { print("Результат:", $0) }
    )
    .store(in: &cancellables)
```

#### 4. tryMap + mapError (очень популярная комбинация)

```swift
$inputText
    .tryMap { text -> Int in
        guard let number = Int(text) else {
            throw ValidationError.invalidNumber
        }
        return number
    }
    .mapError { error -> AppError in
        if let validationError = error as? ValidationError {
            return AppError.validation(validationError)
        }
        return AppError.unknown
    }
    .sink { [weak self] number in
        self?.processNumber(number)
    } onFailure: { error in
        print("Ошибка валидации:", error)
    }
    .store(in: &cancellables)
```

### Лучшие практики tryMap в Combine 2026

- **Используйте tryMap** вместо `map`, если внутри может произойти ошибка (throw)  
- **Всегда** указывайте конкретный тип ошибки в `Future` / `Publisher`, чтобы `tryMap` мог её выбросить  
- **Комбинируйте** с `.mapError`, `.catch`, `.replaceError` — чтобы обработать ошибку дальше по цепочке  
- **Для UI** — после `tryMap` ставьте `.receive(on: DispatchQueue.main)` перед `.sink` / `.assign`  
- **Для сетевых ответов** — `tryMap` + `decode` — классическая пара  
- **В тестах** — используйте `XCTestExpectation` внутри `sink` для проверки ошибки  
- **Документируйте** — всегда пиши комментарий:

```swift
// Преобразование строки в Int с обработкой ошибки парсинга
.tryMap { str -> Int in
    guard let number = Int(str) else {
        throw ValidationError.invalidInput("Ожидалось число, получено: \(str)")
    }
    return number
}
```

**Короткий итог 2026**:
> `tryMap` — это `map`, который **может бросить ошибку** и завершить поток с `Failure`.  
> В 2026 году:  
> - самый популярный кейс — парсинг строк, JSON-декодирование, безопасные вычисления  
> - используйте вместо `map`, если преобразование может fail  
> - комбинируйте с `.mapError`, `.catch`, `.retry` для устойчивости  
> - это **must-have** оператор для любого реального пайплайна с внешними данными  
