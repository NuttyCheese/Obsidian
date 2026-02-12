#ios #swift #objective_c #dispatch_table #method_dispatch #runtime #selectors #dynamic_binding #performance #object_oriented
**Dispatch Table** — это структура данных (обычно словарь или массив), которая сопоставляет **ключи** (или индексы) с **замыканиями** / **функциями** для динамического вызова без `if-else` / [[switch]].

### Зачем нужна

- Упрощает динамический вызов кода
- Заменяет длинные цепочки условий
- Удобна для команд, событий, асинхронных задач, плагинов

### Основные варианты в Swift

| Тип таблицы          | Структура данных       | Когда использовать                     | Пример ключа       |
|----------------------|------------------------|----------------------------------------|--------------------|
| По строковому ключу  | `[String: () → Void]`  | Команды, события, обработчики          | `"play"`, `"pause"` |
| По целочисленному индексу | `[(Int) → Void]`     | Номера команд, состояния               | `0`, `1`, `2`      |
| С параметрами        | `[String: (Any) → Void]` | Универсальные обработчики            | `"updateScore"`    |
| Асинхронная          | `[String: () → Void]` + GCD | Фоновые задачи                        | `"fetchData"`      |

### Примеры кода

#### 1. Простая таблица команд (самый частый вариант)

```swift
let commandTable: [String: () → Void] = [
    "start":  { print("▶️ Запуск") },
    "pause":  { print("⏸️ Пауза") },
    "stop":   { print("⏹️ Стоп") },
    "reset":  { print("🔄 Сброс") }
]

func execute(command: String) {
    commandTable[command]?()
}

execute(command: "pause")  // ⏸️ Пауза
```

#### 2. Таблица с параметрами

```swift
let operations: [String: (Int, Int) → Int] = [
    "add": { $0 + $1 },
    "sub": { $0 - $1 },
    "mul": { $0 * $1 },
    "div": { $1 != 0 ? $0 / $1 : 0 }
]

let result = operations["mul"]?(6, 7)  // 42
```

#### 3. Асинхронная dispatch table ([[GCD]])

```swift
let backgroundTasks: [String: () → Void] = [
    "fetch":  { print("Загрузка данных…") },
    "upload": { print("Выгрузка…") },
    "sync":   { print("Синхронизация…") }
]

func runBackground(task: String) {
    DispatchQueue.global(qos: .utility).async {
        backgroundTasks[task]?()
    }
}

runBackground(task: "fetch")
```

#### 4. Таблица методов класса ([[Objective-C]] style)

```swift
class Player {
    func play()    { print("▶️ Play") }
    func pause()   { print("⏸️ Pause") }
    func stop()    { print("⏹️ Stop") }
}

let player = Player()

let playerCommands: [String: () → Void] = [
    "play":  player.play,
    "pause": player.pause,
    "stop":  player.stop
]

playerCommands["play"]?()  // ▶️ Play
```

#### 5. Безопасный вызов с параметрами и возвратом

```swift
let handlers: [String: (String) → String?] = [
    "greet": { "Привет, \($0)!" },
    "farewell": { "Пока, \($0)!" }
]

if let result = handlers["greet"]?("Анна") {
    print(result)  // Привет, Анна!
}
```

### Краткий итог

- **Dispatch Table** = словарь/массив → замыкание/функция
- Самый удобный формат в Swift: `[String: () → Void]` или `[String: (…Args) → …Return]`
- Заменяет `switch` / `if-else` в сценариях с динамическими командами
- Идеально сочетается с GCD, [[NotificationCenter]], делегатами, плагинами
- В 2026 году — один из самых читаемых и расширяемых паттернов

**Рекомендация**:  
Используй dispatch table вместо длинных `switch` — код становится короче, чище и проще для добавления новых команд.

