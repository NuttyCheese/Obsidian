**Command** (Команда) — это **поведенческий паттерн проектирования** из классической «Банды четырёх» (GoF), который превращает **запрос / действие** в самостоятельный объект.

Это позволяет:
- параметризовать объекты разными действиями
- ставить действия в очередь
- логировать операции
- поддерживать отмену (undo)
- выполнять отложенное выполнение

В Swift / iOS 2026 году паттерн **Command** жив и очень активно используется, но его внешний вид сильно изменился благодаря async/await, closures и TCA.

### Классическая структура паттерна Command (GoF)

| Компонент             | Роль в классическом паттерне                              | Актуальность в Swift 2026 |
|-----------------------|-------------------------------------------------------------|-----------------------------|
| **Command**           | Протокол / интерфейс с методом `execute()`                  | Обязателен (чаще всего протокол) |
| **ConcreteCommand**   | Конкретная реализация команды (сохраняет receiver и параметры) | struct / class / actor / closure |
| **Receiver**          | Объект, который реально выполняет действие                  | Обычно сервис / модель / view model |
| **Invoker**           | Тот, кто вызывает команду (кнопка, меню, очередь)           | UIButton action, TCA reducer, UndoManager |
| **Client**            | Тот, кто создаёт команды и передаёт их invoker’у            | View / ViewModel / Coordinator |

### Самые популярные реализации Command в Swift 2026

#### Вариант 1 — Классический Command (редко, но встречается в legacy и играх)

```swift
protocol Command {
    func execute()
    func undo()
}

class LightOnCommand: Command {
    private let light: Light
    
    init(light: Light) {
        self.light = light
    }
    
    func execute() {
        light.turnOn()
    }
    
    func undo() {
        light.turnOff()
    }
}

class LightOffCommand: Command {
    private let light: Light
    
    init(light: Light) {
        self.light = light
    }
    
    func execute() {
        light.turnOff()
    }
    
    func undo() {
        light.turnOn()
    }
}

class RemoteControl {
    private var command: Command?
    
    func pressButton() {
        command?.execute()
    }
    
    func pressUndo() {
        command?.undo()
    }
    
    func setCommand(_ command: Command) {
        self.command = command
    }
}
```

#### Вариант 2 — Самый популярный в 2026 — **замыкание как команда** (closure-based Command)

```swift
typealias Command = () -> Void

class RemoteControl {
    private var onCommand: Command?
    private var offCommand: Command?
    private var undoCommand: Command?
    
    func setOnCommand(_ command: @escaping Command) {
        onCommand = command
    }
    
    func setOffCommand(_ command: @escaping Command) {
        offCommand = command
    }
    
    func pressOn() {
        onCommand?()
        undoCommand = offCommand
    }
    
    func pressOff() {
        offCommand?()
        undoCommand = onCommand
    }
    
    func pressUndo() {
        undoCommand?()
    }
}

// Использование
let light = Light()
let remote = RemoteControl()

remote.setOnCommand { light.turnOn() }
remote.setOffCommand { light.turnOff() }

remote.pressOn()   // свет включён
remote.pressUndo() // свет выключен
```

**Почему это доминирует в 2026**:
- Минимум boilerplate  
- Полная поддержка async/await  
- Легко передавать параметры через capture  
- Идеально для SwiftUI / TCA / Combine

#### Вариант 3 — Command в **TCA** (The Composable Architecture) — самый современный стиль

```swift
@Reducer
struct LightFeature {
    @ObservableState
    struct State {
        var isOn = false
    }
    
    enum Action {
        case turnOn
        case turnOff
        case toggle
    }
    
    var body: some Reducer<State, Action> {
        Reduce { state, action in
            switch action {
            case .turnOn:
                state.isOn = true
                return .none
                
            case .turnOff:
                state.isOn = false
                return .none
                
            case .toggle:
                state.isOn.toggle()
                return .none
            }
        }
    }
}
```

Здесь **каждый Action** — это по сути **команда**, а **reducer** — invoker.

### Когда Command действительно нужен в 2026 году

| Сценарий                                      | Лучший способ реализации Command | Почему |
|-----------------------------------------------|-----------------------------------|--------|
| Отмена операций (undo/redo)                   | Command + Memento                 | Классический сценарий |
| Очередь задач / макросы                       | Массив Command + executeAll()     | Редко, чаще OperationQueue / TaskGroup |
| TCA / unidirectional flow                     | Action как Command                | Стандарт TCA |
| SwiftUI Button / gesture action               | Closure как Command               | Самый частый случай |
| Кастомный undo manager                        | Command + undo stack              | Редко, чаще NSUndoManager |
| Middleware / декораторы команд                | Command + wrapper                 | Очень мощно |

### Лучшие практики Command в Swift 2026

- **Предпочитай замыкания** (`() -> Void` / `() async throws -> Void`)  
- **Не создавай отдельные классы**, если команда простая  
- **Async команды** — используй `() async throws -> Void`  
- **Отмена** — передавай `Task` или `CancellationToken`  
- **TCA** — используй встроенные Action / Reducer вместо ручного Command  
- **Undo/Redo** — чаще всего достаточно `NSUndoManager` или `UndoManager` в SwiftUI  
- **Swift 6 strict concurrency** — все команды должны быть Sendable или выполняться в правильном actor-контексте  
- **Тестирование** — моки через протокол или замыкания — очень просто

**Короткий девиз 2026**:
> «Command в 2026 году — это когда ты хочешь превратить действие в объект, чтобы его можно было отложить, отменить, залогировать или поставить в очередь.  
> В современном Swift чаще всего это просто замыкание или Action в TCA.  
> Классические классы Command почти не используются — их заменили более лёгкие и мощные конструкции.»

Удачи с предсказуемыми, отменяемыми и тестируемыми действиями в Swift! 🔄