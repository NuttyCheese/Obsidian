Монада — это **контейнер** (или контекст), который умеет:

1. **упаковывать** обычное значение в себя (`pure` / `unit` / `return`)  
2. **распаковывать** значение, применять к нему функцию, которая **сама возвращает монаду**, и снова упаковывать результат (`flatMap` / `bind`)

Главные операторы монады:

- `pure` / `unit` — кладёт значение в монаду  
- `flatMap` — берёт значение из монады → применяет функцию → получает новую монаду

**Коротко и по-человечески**:

- Functor (`map`): «Достань значение из коробки → примени функцию → положи результат обратно в коробку»  
- Applicative (`<*>`): «В одной коробке функция, в другой значение → достань оба → примени → положи результат в коробку»  
- Monad (`flatMap`): «В коробке значение → достань → примени функцию, которая сама вернёт коробку → получи новую коробку»

### 2. Три закона монады (математическая основа)

Чтобы тип считался монадой, он должен удовлетворять трём законам:

1. **Left identity** (левая единица):  
   `pure(x).flatMap(f) == f(x)`

2. **Right identity** (правая единица):  
   `m.flatMap(pure) == m`

3. **Associativity** (ассоциативность):  
   `m.flatMap(f).flatMap(g) == m.flatMap { x in f(x).flatMap(g) }`

Swift не проверяет эти законы на уровне компилятора, но их соблюдение критично для предсказуемого поведения.

### 3. Классические монады в стандартной библиотеке Swift

| Тип                        | `pure` / `unit`      | `flatMap` / `bind`                           | Пример использования                     |
| -------------------------- | -------------------- | -------------------------------------------- | ---------------------------------------- |
| [[Optional]]`<T>`          | `Optional.some(_:)`  | [[flatMap]]                                  | Обработка возможного отсутствия значения |
| `Result<Success, Failure>` | `Result.success(_:)` | `flatMap`                                    | Обработка успеха/ошибки                  |
| [[Array]]`<T>`             | `Array.init(_:)`     | `flatMap`                                    | Работа с множеством значений             |
| `Publisher` ([[Combine]])  | `Just(_:)`           | `flatMap`                                    | Асинхронные потоки данных                |
| [[Task]] / [[async]]       | `Task { ... }`       | `async let` / `flatMap`-подобные конструкции | Асинхронные вычисления                   |

### 4. Полные примеры кода

#### Пример 1 — [[Optional]] как монада (самый частый случай)

```swift
let number: Int? = 5

let result = number
    .flatMap { x in Optional(x + 10) }     // 15
    .flatMap { y in Optional(y * 2) }      // 30
    .flatMap { z in Optional(String(z)) }  // "30"

print(result) // Optional("30")
```

#### Пример 2 — Result как монада (обработка ошибок)

```swift
enum NetworkError: Error { case invalidURL, decodingFailed }

func fetchUserID() -> Result<Int, NetworkError> { .success(42) }
func fetchProfile(id: Int) -> Result<String, NetworkError> { .success("Anna") }

let profile = fetchUserID()
    .flatMap { id in fetchProfile(id: id) }

switch profile {
case .success(let name): print("Привет, \(name)!")
case .failure(let error): print("Ошибка: \(error)")
}
```

#### Пример 3 — Кастомная монада Box (для понимания)

```swift
struct Box<A> {
    let value: A
    
    init(_ value: A) { self.value = value }
    
    func map<B>(_ f: (A) -> B) -> Box<B> {
        Box(f(value))
    }
    
    func flatMap<B>(_ f: (A) -> Box<B>) -> Box<B> {
        f(value)
    }
}

let box = Box(5)
    .flatMap { x in Box(x + 10) }
    .flatMap { y in Box(y * 2) }

print(box.value) // 30
```

#### Пример 4 — IO Monad (классика монад для побочных эффектов)

```swift
struct IO<A> {
    let run: () throws -> A
    
    init(_ run: @escaping () throws -> A) { self.run = run }
    
    static func pure(_ value: A) -> IO<A> { IO { value } }
    
    func map<B>(_ f: @escaping (A) throws -> B) rethrows -> IO<B> {
        IO { try f(self.run()) }
    }
    
    func flatMap<B>(_ f: @escaping (A) throws -> IO<B>) rethrows -> IO<B> {
        IO { try f(self.run()).run() }
    }
}

let readName = IO<String> {
    print("Введите имя: ", terminator: "")
    return readLine() ?? "Anonymous"
}

let greet = readName.flatMap { name in
    IO { print("Привет, \(name)!") }
}

try greet.run()
```

#### Пример 5 — Асинхронный IO (2026 стандарт)

```swift
struct AsyncIO<A> {
    let run: () async throws -> A
    
    init(_ run: @escaping () async throws -> A) { self.run = run }
    
    func flatMap<B>(_ f: @escaping (A) async throws -> AsyncIO<B>) async rethrows -> AsyncIO<B> {
        AsyncIO {
            try await f(self.run()).run()
        }
    }
}

let fetchUser = AsyncIO { try await URLSession.shared.data(from: URL(string: "https://api.example.com/user")!) }

let program = fetchUser.flatMap { data in
    AsyncIO { print("Получены данные: \(data.count) байт") }
}

await program.run()
```

### 6. Сравнение Functor vs Applicative vs Monad (2026)

| Концепция          | Оператор | Что позволяет                              | Пример в Swift |
|--------------------|----------|--------------------------------------------|----------------|
| Functor            | `map`    | f(a) → F<b>                                | `Optional.map` |
| Applicative        | `<*>`    | F<(a → b)> <*> F<a> → F<b>                 | `<*>` для Optional/Result |
| Monad              | `flatMap`| F<a> → (a → F<b>) → F<b>                   | `Optional.flatMap`, `Result.flatMap` |

### 7. Лучшие практики 2026

- Используйте **монады** там, где есть **побочные эффекты** или **асинхронность** (IO, Result, [[Combine]], [[async]]/[[await]])  
- **Не злоупотребляйте** глубокими цепочками `flatMap` — лучше `async let` / `TaskGroup` для параллелизма  
- Для тестов — создавайте **Mock монады** (MockIO, TestResult и т.д.)  
- В архитектуре — **Functional Core, Imperative Shell**: чистая логика в монадах, запуск в оболочке  
- Для сложных приложений — используйте библиотеки:  
  - **Bow** (Functional Swift)  
  - **swift-dependencies** + **swift-case-paths**  
  - **The Composable Architecture (TCA)** от Point-Free  
- В [[Swift]] 6 — монады помогают с **strict concurrency** (явно показывают, где побочные эффекты)

**Короткий девиз 2026**:
> «Монада — это когда ты говоришь: «Я хочу сделать что-то грязное (побочный эффект), но весь остальной код останется чистым».  
> flatMap — это цепочка безопасных шагов.  
> run() — это точка, где ты разрешаешь миру измениться.»
