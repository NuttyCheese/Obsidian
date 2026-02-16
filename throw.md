**throw** в [[Swift]] — это ключевое слово, которое используется для **генерации (выбрасывания) ошибки** в функциях, методах и замыканиях, помеченных как `throws` или `async throws`.

Это основной механизм **обработки ошибок** в современной Swift-программировании (особенно с [[async]]/[[await]] и [[Swift]] 6 strict concurrency).

### Основные правила и синтаксис (2026 актуальные)

1. **Функция / метод может бросать ошибку** — помечается `throws` или `async throws`

```swift
func divide(_ a: Int, by b: Int) throws -> Int {
    guard b != 0 else {
        throw DivisionError.zeroDivision
    }
    return a / b
}
```

2. **Вызов бросающей функции** — требует `try`, `try?` или `try!`

```swift
// Вариант 1: try + do-catch (самый рекомендуемый)
do {
    let result = try divide(10, by: 0)
    print(result)
} catch {
    print("Ошибка: \(error)")
}

// Вариант 2: try? — возвращает Optional
let result1 = try? divide(10, by: 2)  // 5
let result2 = try? divide(10, by: 0)  // nil

// Вариант 3: try! — force try (краш при ошибке)
let result = try! divide(10, by: 2)   // 5
// try! divide(10, by: 0)             // runtime crash
```

3. **async throws** — в асинхронных функциях

```swift
enum NetworkError: Error {
    case invalidURL
    case noConnection
}

func fetchData(from urlString: String) async throws -> Data {
    guard let url = URL(string: urlString) else {
        throw NetworkError.invalidURL
    }
    
    let (data, _) = try await URLSession.shared.data(from: url)
    return data
}

// Вызов
Task {
    do {
        let data = try await fetchData(from: "https://api.example.com")
        print("Данные получены")
    } catch {
        print("Сетевая ошибка: \(error)")
    }
}
```

### Типы ошибок (Error) в 2026 году

| Тип ошибки                  | Когда использовать | Пример |
|-----------------------------|--------------------|--------|
| Перечисление enum           | Самый частый и рекомендуемый | `enum NetworkError: Error { case invalidURL, timeout }` |
| Структурированный тип       | Когда нужна дополнительная информация | `struct APIError: Error { let code: Int; let message: String }` |
| LocalizedError              | Для пользовательских сообщений | `enum LoginError: LocalizedError { var errorDescription: String? { ... } }` |
| CustomNSError               | Редко, для ObjC-совместимости | `NSError(domain: "com.app", code: -1001, userInfo: [...])` |
| Never                       | Для функций, которые никогда не бросают ошибок | `throws(Never)` (редко) |

### Лучшие практики throw / throws в Swift 2026

- **throws** — всегда явно указывай в сигнатуре функции/метода  
- **async throws** — стандарт для сетевых, файловых, БД-операций  
- **do-try-catch** — основной способ обработки ошибок в продакшене  
- **try?** — для случаев, когда ошибка не критична (fallback на [[nil]])  
- **try!** — только в тестах / уверенных случаях (краш при ошибке — это нормально в тестах)  
- **Ошибки как enum** — предпочтительнее [[struct]] / [[class]] (ассоциированные значения + LocalizedError)  
- **Swift 6 strict concurrency** — throws-функции безопасны в [[actor]] / [[Task]], если Error [[Sendable]]  
- **Ошибки в протоколах** — `throws` в протоколах требует, чтобы реализация тоже бросала ошибку  
- **Документируйте** — пиши `/// throws NetworkError.invalidURL если URL некорректен`

**Короткий девиз 2026**:
> «throw — это когда ты говоришь: «здесь может произойти ошибка, и вызывающий код обязан её обработать».  
> В 2026 году throws + async throws + enum Error — это **стандарт** для всей асинхронной и потенциально опасной логики в Swift.  
> Не прячь ошибки — пусть компилятор заставляет их обрабатывать.»
