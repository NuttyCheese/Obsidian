**State Monad** — это монада, которая позволяет **управлять изменяемым состоянием** в чисто функциональном стиле, не используя мутацию ([[var]]) и глобальные переменные.

Она инкапсулирует пару:

- **состояние** (State)  
- **результат вычисления** (Value)

Каждый шаг программы получает текущее состояние → выполняет вычисление → возвращает **новое состояние** и **результат**.

**Главные преимущества State Monad**:

- Состояние **явно** передаётся через тип (нет глобальных переменных)  
- Легко **тестировать** (передаём начальное состояние → получаем результат и финальное состояние)  
- Код остаётся **чистым** и **композируемым**  
- Нет [[race condition]] (если комбинировать с Reader / IO)  
- Идеально сочетается с **[[async]]/[[await]]** и **[[Combine]]**

### 2. Классическая реализация State Monad в [[Swift]]

```swift
struct State<S, A> {
    // Основная функция: принимает старое состояние → возвращает (результат, новое состояние)
    let run: (S) -> (A, S)
    
    init(_ run: @escaping (S) -> (A, S)) {
        self.run = run
    }
    
    // unit / pure / return — упаковка значения без изменения состояния
    static func pure(_ value: A) -> State<S, A> {
        State { state in (value, state) }
    }
    
    // map — преобразование результата (Functor)
    func map<B>(_ f: @escaping (A) -> B) -> State<S, B> {
        State { state in
            let (value, newState) = self.run(state)
            return (f(value), newState)
        }
    }
    
    // flatMap — композиция монад (Monad)
    func flatMap<B>(_ f: @escaping (A) -> State<S, B>) -> State<S, B> {
        State { state in
            let (value, newState) = self.run(state)
            return f(value).run(newState)
        }
    }
    
    // Получить текущее состояние (ask)
    static func get() -> State<S, S> {
        State { state in (state, state) }
    }
    
    // Установить новое состояние (put)
    static func put(_ newState: S) -> State<S, Void> {
        State { _ in ((), newState) }
    }
    
    // Изменить текущее состояние (modify)
    static func modify(_ f: @escaping (S) -> S) -> State<S, Void> {
        State { state in ((), f(state)) }
    }
    
    // Выполнить и получить результат + финальное состояние
    func run(_ initialState: S) -> (A, S) {
        run(initialState)
    }
}
```

### 3. Простые и понятные примеры

#### Пример 1 — Счётчик (классика)

```swift
// Инкремент счётчика
let increment = State<Int, Int>.get()           // получаем текущее значение
    .flatMap { current in
        State.put(current + 1)                  // увеличиваем
            .map { _ in current + 1 }           // возвращаем новое значение
    }

let (result, finalState) = increment.run(0)
print(result, finalState)      // 1 1

// Два инкремента подряд
let twoIncrements = increment.flatMap { _ in increment }
let (res2, state2) = twoIncrements.run(0)
print(res2, state2)            // 2 2
```

#### Пример 2 — Генератор случайных чисел (очень популярный кейс)

```swift
import Foundation

typealias RNG = UInt64

func randomUInt64() -> State<RNG, UInt64> {
    State { seed in
        // Простой линейный конгруэнтный генератор
        let newSeed = seed &* 6364136223846793005 &+ 1442695040888963407
        let value = newSeed >> 32
        return (value, newSeed)
    }
}

let randomPair = randomUInt64()
    .flatMap { first in
        randomUInt64().map { second in (first, second) }
    }

let (pair, finalSeed) = randomPair.run(12345)
print("Случайная пара:", pair, "новый seed:", finalSeed)
```

#### Пример 3 — Стек (push / pop)

```swift
typealias StackState<A> = State<[A], A>

let push = { (value: A) -> StackState<A> in
    State.modify { stack in stack + [value] }
        .map { _ in value }
}

let pop: StackState<A> = State { stack in
    guard let last = stack.last else { fatalError("Stack empty") }
    return (last, Array(stack.dropLast()))
}

let program = push(10)
    .flatMap { _ in push(20) }
    .flatMap { _ in pop }
    .flatMap { top in pop.map { second in (top, second) } }

let (result, finalStack) = program.run([])
print("Результат:", result)     // (20, 10)
print("Финальный стек:", finalStack)  // []
```

#### Пример 4 — Асинхронный State (2026 стандарт)

```swift
struct AsyncState<S, A> {
    let run: (S) async -> (A, S)
    
    func flatMap<B>(_ f: @escaping (A) async -> AsyncState<S, B>) async -> AsyncState<S, B> {
        AsyncState { state in
            let (value, newState) = await self.run(state)
            return await f(value).run(newState)
        }
    }
}

func asyncIncrement() -> AsyncState<Int, Int> {
    AsyncState { state in
        try? await Task.sleep(nanoseconds: 1_000_000_000)
        return (state + 1, state + 1)
    }
}
```

### 5. Сравнение State Monad с другими подходами

| Подход                                | Управление состоянием | Чистота функций | Тестируемость | Композиция | Сложность |
| ------------------------------------- | --------------------- | --------------- | ------------- | ---------- | --------- |
| Глобальные var / Singleton            | Глобально             | Низкая          | Плохая        | Плохо      | Низкая    |
| Передача состояния вручную            | Явно в параметрах     | Средняя         | Хорошая       | Средняя    | Средняя   |
| State Monad                           | Через State<S, A>     | Высокая         | Отличная      | Отлично    | Средняя   |
| [[Swift]] 6 @Observable / Observation | Автоматически         | Средняя         | Средняя       | Хорошая    | Низкая    |
| TCA (Composable Architecture)         | Через Reducer/State   | Высокая         | Отличная      | Отлично    | Высокая   |

### 6. Лучшие практики 2026

- Используйте **AsyncState** для современных асинхронных приложений  
- **Не выполняйте** `run()` внутри чистых функций — только в оболочке (App, ViewModel, [[SceneDelegate]])  
- Для тестов — передавайте начальное состояние и проверяйте результат + финальное состояние  
- Комбинируйте с **Reader Monad** (ReaderT / Environment) для передачи зависимостей  
- Для больших приложений — рассмотрите **The Composable Architecture (TCA)** или **swift-dependencies** вместо ручного State  
- Избегайте глубоких цепочек `flatMap` — лучше разбивать на маленькие функции  
- Для параллельных операций — используйте `async let` / `TaskGroup` вместо последовательного flatMap

**Короткий девиз 2026**:
> «State Monad — это когда ты говоришь: «Я хочу менять состояние, но делать это чисто, предсказуемо и тестируемо».  
> flatMap — это безопасный шаг: старое состояние → новое состояние + результат.  
> run() — это точка, где ты разрешаешь состоянию измениться.»
