### 1. Что такое IO Monad и зачем она нужна

**IO Monad** — это монада, которая **инкапсулирует побочные эффекты** (side effects), такие как:

- чтение/запись файлов  
- ввод/вывод в консоль  
- сетевые запросы  
- работа с датчиками устройства  
- изменение глобального состояния  
- вызов внешних [[API]] и т.д.

Главный принцип функционального программирования: **чистые функции** не должны иметь побочных эффектов.  
IO Monad позволяет **отложить** выполнение побочных эффектов и **явно** показать в типе, что функция может изменить внешний мир.

**Коротко и по-человечески**:

- Обычная функция: `Int →` [[Int]] — чистая, предсказуемая  
- Функция с побочным эффектом: `Int → Int` (но внутри пишет в файл) — нечистая  
- С IO Monad: `Int → IO<Int>` — честно говорит: «я верну число, но могу напакостить во внешнем мире»

### 2. Классическая структура IO Monad в [[Swift]]

```swift
struct IO<A> {
    let run: () throws -> A   // или () async throws -> A в 2026 году
    
    init(_ run: @escaping () throws -> A) {
        self.run = run
    }
    
    // unit / pure
    static func pure(_ value: A) -> IO<A> {
        IO { value }
    }
    
    // map (Functor)
    func map<B>(_ f: @escaping (A) throws -> B) rethrows -> IO<B> {
        IO {
            try f(self.run())
        }
    }
    
    // flatMap (Monad)
    func flatMap<B>(_ f: @escaping (A) throws -> IO<B>) rethrows -> IO<B> {
        IO {
            try f(self.run()).run()
        }
    }
    
    // apply (Applicative)
    static func <*> <B>(f: IO<(A) throws -> B>, a: IO<A>) rethrows -> IO<B> {
        IO {
            try f.run()(a.run())
        }
    }
}
```

**Современный вариант 2026** (с [[async]]/[[await]]):

```swift
struct AsyncIO<A> {
    let run: () async throws -> A
    
    init(_ run: @escaping () async throws -> A) {
        self.run = run
    }
    
    static func pure(_ value: A) -> AsyncIO<A> {
        AsyncIO { value }
    }
    
    func map<B>(_ f: @escaping (A) async throws -> B) async rethrows -> AsyncIO<B> {
        AsyncIO {
            try await f(self.run())
        }
    }
    
    func flatMap<B>(_ f: @escaping (A) async throws -> AsyncIO<B>) async rethrows -> AsyncIO<B> {
        AsyncIO {
            try await f(self.run()).run()
        }
    }
}
```

### 3. Простые примеры

#### Пример 1 — Консольный ввод/вывод

```swift
let printLine = IO { message in
    print(message)
}

let readLineIO = IO<String> {
    print("> ", terminator: "")
    return readLine() ?? ""
}

let program = printLine("Привет! Как тебя зовут?")
    .flatMap { _ in readLineIO }
    .flatMap { name in
        printLine("Приятно познакомиться, \(name)!")
    }

program.run()  // выполнит всю цепочку
```

#### Пример 2 — Работа с файлами (классика IO Monad)

```swift
func readFile(_ path: String) -> IO<String> {
    IO {
        try String(contentsOfFile: path, encoding: .utf8)
    }
}

func writeFile(_ path: String, content: String) -> IO<Void> {
    IO {
        try content.write(toFile: path, atomically: true, encoding: .utf8)
    }
}

let fileProgram = readFile("input.txt")
    .map { $0.uppercased() }
    .flatMap { upper in writeFile("output.txt", content: upper) }

fileProgram.run()  // прочитает, преобразует, запишет
```

#### Пример 3 — Асинхронный IO (2026 стандарт)

```swift
func fetchUser(id: Int) async throws -> User {
    // сетевой запрос
}

let fetchIO = AsyncIO { try await fetchUser(id: 42) }

let program = fetchIO
    .map { user in user.name.uppercased() }
    .flatMap { name in
        AsyncIO { print("Привет, \(name)!") }
    }

await program.run()
```

### 4. Сравнение Functor vs Applicative vs Monad vs IO

| Концепция   | Оператор    | Что позволяет                         | Пример в Swift     | IO Monad?                 |
| ----------- | ----------- | ------------------------------------- | ------------------ | ------------------------- |
| Functor     | [[map]]     | f(a) → F<b>                           | `Optional.map`     | Да                        |
| Applicative | `<*>`       | F<(a → b)> <*> F<a> → F<b>            | `<*>` для Optional | Да                        |
| Monad       | [[flatMap]] | F<a> → (a → F<b>) → F<b>              | `flatMap`          | Да                        |
| IO Monad    | `run`       | Выполняет отложенные побочные эффекты | `IO.run()`         | Специализированная монада |

### 5. Реальные сценарии в iOS-разработке 2026

1. **Консольные утилиты / CLI-инструменты** — ввод/вывод через IO  
2. **Тестирование** — подмена IO на мок (mock IO)  
3. **Архитектура** — чистая бизнес-логика без побочных эффектов  
4. **Асинхронные цепочки** — async/await + IO для сетевых/файловых операций  
5. **Functional Core, Imperative Shell** — бизнес-логика чистая, оболочка в IO  
6. **[[SwiftUI]] + IO** — ViewModel возвращает `IO<Action>` для обработки событий

### 6. Лучшие практики 2026

- Используйте **AsyncIO** с `async throws` — современный стандарт  
- **Не выполняйте** `run()` внутри чистых функций — только в оболочке (main, [[AppDelegate]], [[SceneDelegate]])  
- Для тестов — создавайте **MockIO** или **TestIO**  
- Комбинируйте с **Reader Monad** / **Environment** для передачи зависимостей  
- Для сложных программ — используйте библиотеки:  
  - Bow (Functional [[Swift]])  
  - swift-case-paths + swift-dependencies  
  - Point-Free’s swift-composable-architecture (TCA)  
- Избегайте глубоких цепочек `flatMap` — лучше `async let` / `TaskGroup` для параллелизма  
- Для простых случаев — достаточно `async throws` без явной IO Monad

**Короткий девиз 2026**:
> «IO Monad — это коробка, в которую мы кладём всё грязное и опасное: файлы, сеть, консоль.  
> Остальной код остаётся чистым и предсказуемым.  
> Запускай коробку только в одном месте — и мир будет в порядке.»
