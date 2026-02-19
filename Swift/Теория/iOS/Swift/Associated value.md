**Associated Value** (ассоциированные значения) — это одна из самых мощных и часто недооценённых фич [[Swift]]-[[enum]]. Благодаря им перечисление превращается из простого списка вариантов в полноценную **типобезопасную модель данных** с параметрами.

В 2026 году это **основной** инструмент для моделирования состояний, результатов операций, ошибок, событий и вариантов поведения. Почти все серьёзные Swift-приложения используют `enum` с associated values.

### 1. Почему associated values — это круто (2026 взгляд)

| Задача / Сценарий                             | Без associated values (плохо)                     | С associated values (правильно)                     | Преимущества |
|-----------------------------------------------|----------------------------------------------------|------------------------------------------------------|--------------|
| Результат сетевого запроса                    | `enum NetworkResult { case success, failure }`     | `case success(statusCode: Int, data: Data)`          | Получаем код ответа и данные |
| Ошибка с описанием                            | `case error`                                       | `case error(message: String, code: Int)`             | Конкретная информация об ошибке |
| Состояние авторизации                         | `case loggedIn, loggedOut`                         | `case loggedIn(user: User, token: String)`           | Храним пользователя и токен |
| Разные типы событий                           | `case tap, swipe, longPress`                       | `case tap(location: CGPoint), swipe(direction: Direction)` | Параметры события |
| Фигуры / геометрия                            | `case circle, square`                              | `case circle(radius: Double), square(side: Double)`  | Можно считать площадь, периметр |
| Операции / действия                           | `case add, multiply`                               | `case add(a: Int, b: Int), multiply(a: Int, b: Int)` | Можно сразу выполнить операцию |

### 2. Базовый синтаксис и ключевые правила

```swift
enum NetworkResponse {
    case success(statusCode: Int, data: Data)
    case failure(error: Error, retryCount: Int)
    case loading(progress: Double)
    case cancelled
}
```

- Каждый `case` может иметь **свои** ассоциированные значения (или вообще не иметь)
- Типы могут быть **любые**: [[Int]], [[String]], [[Data]], [[Error]], [[struct]], [[class]], [[enum]], [[tuple]], [[Any]], [[any Protocol]] и т.д.
- Значения **задаются при создании** экземпляра enum
- Значения **доступны только через pattern matching** (`switch`, `if case`, `guard case`, `for case`)

### 3. Полный разбор всех способов работы с associated values (2026)

#### 3.1. Создание экземпляра

```swift
let success = NetworkResponse.success(statusCode: 200, data: jsonData)
let error   = NetworkResponse.failure(error: apiError, retryCount: 2)
let loading = NetworkResponse.loading(progress: 0.75)
let cancelled = NetworkResponse.cancelled
```

#### 3.2. Извлечение значений — switch (самый популярный)

```swift
switch response {
case .success(let code, let data):
    print("Успех! Код: \(code), размер данных: \(data.count) байт")
    
case .failure(let error, let retries):
    print("Ошибка: \(error.localizedDescription), попыток осталось: \(retries)")
    
case .loading(let progress):
    print("Загрузка: \(Int(progress * 100))%")
    
case .cancelled:
    print("Запрос отменён")
}
```

#### 3.3. Короткий вариант — if case / guard case

```swift
if case .success(let code, _) = response, code == 200 {
    // обработка успешного ответа
}

guard case .success(_, let data) = response else {
    print("Не успех")
    return
}
// data доступна
```

#### 3.4. for case — фильтрация в цикле

```swift
let responses: [NetworkResponse] = [success, error, loading, success2]

for case .success(let code, _) in responses where code == 200 {
    print("Нашёл успешный ответ с кодом 200")
}
```

#### 3.5. Associated values + enum с методами

```swift
enum Operation {
    case add(a: Int, b: Int)
    case multiply(a: Int, b: Int)
    case divide(a: Double, b: Double)
    
    func perform() -> Double {
        switch self {
        case .add(let a, let b):     return Double(a + b)
        case .multiply(let a, let b): return Double(a * b)
        case .divide(let a, let b):  return b != 0 ? a / b : .nan
        }
    }
}

let op = Operation.add(a: 5, b: 3)
print(op.perform()) // 8.0
```

### 4. Самые полезные паттерны 2026 года

#### 4.1. Result + associated values (самый популярный)

```swift
enum Result<Value, Failure: Error> {
    case success(Value)
    case failure(Failure)
    
    var value: Value? {
        if case .success(let v) = self { return v }
        return nil
    }
    
    var error: Failure? {
        if case .failure(let e) = self { return e }
        return nil
    }
}
```

#### 4.2. Состояние загрузки (Loading, Success, Failure)

```swift
enum LoadState<T> {
    case idle
    case loading
    case success(T)
    case failure(Error)
}
```

#### 4.3. Координаты + тип точки

```swift
enum Point {
    case cartesian(x: Double, y: Double)
    case polar(radius: Double, angle: Double)
}
```

### 5. Лучшие практики associated values в Swift 2026

- **Используйте enum вместо [[class]]/[[struct]]** для моделирования состояний с вариантами  
- **Давайте осмысленные имена** associated values: `userID: String` лучше, чем `value: String`  
- **Используйте tuple** для нескольких значений, если они логически связаны  
- **Добавляйте методы** к enum — `perform()`, `value`, `error`, `isSuccess` и т.д.  
- **Swift 6 strict concurrency** — enum с associated values полностью безопасен, если `Value` и `Failure` — [[Sendable]]  
- **Документируйте** — пиши комментарий «NetworkResponse.success — успешный ответ с кодом и данными»

**Короткий девиз 2026**:
> Associated values — это когда enum становится **не просто списком вариантов**, а **полноценной моделью данных** с параметрами.  
> В 2026 году это **основной** инструмент для Result, LoadState, Operation, Event, Response, Error и любых состояний с разными данными.  
> Используй `switch`, `if case`, `guard case` — и твой код станет типобезопасным, читаемым и мощным.
