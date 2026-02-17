**Selector** в [[Swift]] — это тип данных, который представляет **имя метода** (или функции) объекта. Он позволяет **динамически** ссылаться на метод и вызывать его во время выполнения программы.

В чистом Swift это не очень распространённый тип — Swift предпочитает статическую типизацию и замыкания.  
Но **Selector** остаётся крайне важным, потому что:

- Большая часть [[UIKit]], [[Foundation]] и старого Cocoa построена на [[Objective-C]] [[Runtime]]  
- Многие API UIKit до сих пор используют **target-action** механизм, который требует именно `Selector`  
- Без `Selector` не работают кнопки, жесты, таймеры, уведомления и многое другое

### 1. Что такое Selector на самом деле

`Selector` — это структура, которая хранит **имя метода** в виде строки, понятной Objective-C runtime.

```swift
let sel = #selector(MyClass.methodName(_:))
print(sel)  // → "methodName:"
```

- `#selector(...)` — это специальный синтаксис Swift, который проверяет существование метода на этапе компиляции  
- Без `#selector` — просто строка `"methodName:"` (но тогда нет проверки компилятором)

### 2. Почему метод должен быть помечен `@objc`

Чтобы метод стал видимым для Objective-C runtime (а значит и для `Selector`), он должен быть:

```swift
@objc func buttonPressed(_ sender: UIButton) { ... }
```

Без `@objc` компилятор выдаст ошибку:

```
Argument of '#selector' does not refer to '@objc' method, property, or initializer
```

Исключения (когда `@objc` не нужен явно):
- Метод уже унаследован от `NSObject` и помечен `@objc` в суперклассе
- Метод реализует `@objc` протокол

### 3. Самые частые места использования Selector в 2026 году

| API / Механизм                                | Как используется Selector                     | Пример (коротко)                                                                      |
| --------------------------------------------- | --------------------------------------------- | ------------------------------------------------------------------------------------- |
| `addTarget(_:action:for:)` (UIButton и др.)   | `action: #selector(methodName(_:))`           | `button.addTarget(self, action: #selector(tapped), for: .touchUpInside)`              |
| `Timer.scheduledTimer`                        | `selector: #selector(timerTick)`              | `Timer.scheduledTimer(timeInterval: 1, target: self, selector: #selector(tick), ...)` |
| `NotificationCenter.addObserver`              | `selector: #selector(didReceiveNotification)` | `center.addObserver(self, selector: #selector(handleNotif), name: ..., object: nil)`  |
| `performSelector(onMainThread:)`              | `selector: #selector(updateUI)`               | Редко, но иногда в legacy                                                             |
| [[UIMenu]] / [[UICommand]] (контекстные меню) | `action: #selector(copyText)`                 | `UICommand(title: "Copy", action: #selector(copy))`                                   |
| `UIApplication.sendAction(_:to:from:for:)`    | Прямой вызов через Selector                   | Когда target = nil — ищет по Responder Chain                                          |

### 4. Самый полный и современный пример 2026 года

```swift
final class SettingsViewController: UIViewController {
    
    private let toggleSwitch = UISwitch()
    private let saveButton = UIButton(type: .system)
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        
        // 1. Настройка переключателя
        toggleSwitch.addTarget(self, action: #selector(switchValueChanged), for: .valueChanged)
        
        // 2. Настройка кнопки
        saveButton.setTitle("Сохранить", for: .normal)
        saveButton.addTarget(self, action: #selector(saveTapped), for: .touchUpInside)
        
        // 3. Добавляем в UI
        // ... layout код ...
    }
    
    // Обработчик переключателя
    @objc private func switchValueChanged(_ sender: UISwitch) {
        print("Переключатель: \(sender.isOn ? "Вкл" : "Выкл")")
        UserDefaults.standard.set(sender.isOn, forKey: "darkMode")
    }
    
    // Обработчик кнопки
    @objc private func saveTapped(_ sender: UIButton) {
        print("Сохранение настроек...")
        // Логика сохранения
    }
    
    // Пример вызова через performSelector (редко, но иногда нужно)
    func delayedUpdate() {
        perform(#selector(updateUI), with: nil, afterDelay: 2.0)
    }
    
    @objc private func updateUI() {
        // Обновить UI через 2 секунды
    }
}
```

### 5. Почему Selector до сих пор жив в 2026 году

- **UIKit не переписан на Swift** — он по-прежнему Objective-C под капотом  
- **target-action** — это классический паттерн Cocoa, который не заменён полностью замыканиями  
- **Совместимость** — старые библиотеки, Objective-C код, App Extensions всё ещё используют Selector  
- **Гибкость** — target = [[nil]] + [[Responder Chain]] позволяет писать очень declarative UI

### 6. Альтернативы Selector в современном Swift

| Ситуация          | Selector (традиционно)                     | Современная альтернатива 2026                                           |
| ----------------- | ------------------------------------------ | ----------------------------------------------------------------------- |
| Кнопка нажата     | `addTarget(self, action: #selector(...))`  | `button.addAction(UIAction { _ in ... }, for: .touchUpInside)`          |
| Таймер            | `Timer.scheduledTimer(..., selector: ...)` | `Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { _ in ... }` |
| Уведомления       | `addObserver(self, selector: ...)`         | `NotificationCenter.default.publisher(for: ...).sink { ... }`           |
| Меню ([[UIMenu]]) | `action: #selector(...)`                   | `UIAction(title: "Copy", handler: { _ in ... })`                        |

### Короткий девиз 2026

> Selector — это **имя метода**, которое может вызвать Objective-C runtime.  
> В 2026 году он нужен почти исключительно для **UIKit target-action**, [[Timer]], [[NotificationCenter]] и legacy.  
> Для нового кода — предпочитай **[[closure]]** ([[UIAction]], `Timer.scheduledTimer { ... }`, [[Combine]], [[async]]/[[await]]).  
> Но если видишь `#selector` — знай, что под капотом работает старая, но надёжная магия Cocoa.
