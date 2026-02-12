#swift #ios #method_overloading #function_overloading #polymorphism #parameters #compile_time #type_signature #code_reuse #object_oriented
# Перегрузка методов (Method Overloading) в [[Swift]]

**Перегрузка методов** — это механизм, позволяющий объявлять **несколько методов с одинаковым именем**, но с **разными сигнатурами** (количеством, типами или внешними/внутренними именами параметров).  
Компилятор Swift выбирает нужную версию **на этапе компиляции** (статическая диспетчеризация).

### Основные правила перегрузки в Swift

| Критерий                      | Можно ли перегружать? | Примечание / пример                                                   |
| ----------------------------- | --------------------- | --------------------------------------------------------------------- |
| Количество параметров         | Да                    | `func f(_ a: Int)` и `func f(_ a: Int, _ b: Int)`                     |
| Типы параметров               | Да                    | `func add(_ a: Int, _ b: Int)` и `func add(_ a: Double, _ b: Double)` |
| Внешние имена параметров      | Да                    | `func greet(name: String)` и `func greet(person name: String)`        |
| Возвращаемый тип              | Нет (сам по себе)     | Возвращаемый тип **не участвует** в сигнатуре                         |
| [[Generic]] vs конкретный тип | Да                    | `func print<T>(_ value: T)` и `func print(_ value: String)`           |
| `throws` vs non-throwing      | Да                    | `func load() throws` и `func load() -> Result`                        |
| [[async]] vs [[sync]]         | Да (Swift 5.5+)       | `func fetch()` и `func fetch() async`                                 |
| [[inout]] vs обычный параметр | Да                    | `func swap(_ a: inout Int, _ b: inout Int)`                           |

### Примеры перегрузки

#### 1. Классическая перегрузка по типу и количеству

```swift
struct Math {
    func add(_ a: Int, _ b: Int) -> Int {
        a + b
    }
    
    func add(_ a: Double, _ b: Double) -> Double {
        a + b
    }
    
    func add(_ a: String, _ b: String) -> String {
        a + b
    }
    
    func add(_ numbers: Int...) -> Int {
        numbers.reduce(0, +)
    }
}

let m = Math()
m.add(3, 4)           // → 7 (Int)
m.add(3.5, 2.5)       // → 6.0 (Double)
m.add("Hello, ", "world")  // → "Hello, world"
m.add(1, 2, 3, 4)     // → 10 (variadic)
```

#### 2. Перегрузка по внешним именам параметров

```swift
struct Logger {
    func log(message: String) {
        print("[INFO] \(message)")
    }
    
    func log(error message: String) {
        print("[ERROR] \(message)")
    }
    
    func log(_ message: String, level: String = "DEBUG") {
        print("[\(level)] \(message)")
    }
}

let logger = Logger()
logger.log(message: "Starting app")         // [INFO] Starting app
logger.log(error: "Failed to connect")      // [ERROR] Failed to connect
logger.log("Retry attempt", level: "WARN")  // [WARN] Retry attempt
```

#### 3. Перегрузка с generics

```swift
struct Printer {
    func print<T>(_ value: T) {
        print("Generic: \(value)")
    }
    
    func print(_ string: String) {
        print("String: \(string.uppercased())")
    }
}

let p = Printer()
p.print(42)             // Generic: 42
p.print("hello")        // String: HELLO
```

#### 4. Перегрузка async / [[throws]]

```swift
class NetworkService {
    func fetch() throws -> Data { /* ... */ }
    func fetch() async throws -> Data { /* ... */ }
    
    func fetch(completion: @escaping (Result<Data, Error>) -> Void) {
        // ...
    }
}
```

### Важные особенности и ограничения

- **Возвращаемый тип не влияет** на выбор метода  
  ```swift
  func get() -> Int { 1 }
  func get() -> String { "one" }   // Ошибка компиляции!
  ```

- Перегрузка **не работает** с изменением только `throws` / `async` без изменения параметров  
  (нужен другой способ отличить сигнатуры)

- **Перегрузка vs переопределение ([[override]])**  
  Перегрузка — **разные сигнатуры**, один класс  
  Переопределение — **та же сигнатура**, в подклассе

### Практические рекомендации (2026)

- Используй перегрузку, когда методы **логически похожи**, но работают с разными типами/количеством аргументов
- Не злоупотребляй — слишком много перегрузок одного имени ухудшают читаемость
- Для различия поведения лучше использовать **разные имена** методов
- В generics + протоколах чаще используй **[[AssociatedType]]** и **constraints**, чем перегрузку

**Короткое правило**:
> «Перегружай методы, когда это делает [[API]] **интуитивным** и **типобезопасным**.  
> Если перегрузка вызывает путаницу — лучше дать разным методам **разные имена**.»
