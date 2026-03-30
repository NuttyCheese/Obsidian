**`self`** в [[Swift]] — это ключевое слово, которое **всегда** ссылается на **текущий экземпляр** типа (класса, структуры, перечисления или актёра), внутри которого вы находитесь.

Это аналог `this` в Java/C++/C# или `Me` в некоторых других языках, но в Swift у `self` есть несколько важных нюансов и очень строгая семантика, особенно после Swift 5.3+ и строгой конкурентности в Swift 6.

### 1. Самые частые случаи использования self (и почему оно нужно)

| Ситуация                                      | Нужно ли писать self?    | Пример (с self и без)                     | Почему self обязателен здесь                         |
| --------------------------------------------- | ------------------------ | ----------------------------------------- | ---------------------------------------------------- |
| Имя параметра совпадает с именем свойства     | **Да**                   | `init(name: String) { self.name = name }` | Иначе компилятор подумает, что `name` — это параметр |
| Внутри замыкания, которое захватывает self    | **Да** (или [weak self]) | `{ print(self.name) }`                    | Замыкание требует явного self ([[Capture list]])     |
| Вызов метода или свойства в обычном методе    | **Нет** (но можно)       | `print(name)` или `print(self.name)`      | Swift позволяет опускать self почти всегда           |
| Передача текущего экземпляра в функцию        | **Да**                   | `someFunction(self)`                      | Без self компилятор не поймёт, что передавать        |
| Явное указание типа при неоднозначности       | **Да**                   | `self as? MyProtocol`                     | Для приведения типов или generic контекста           |
| В статических методах / [[enum]] / [[struct]] | **Нет**                  | `static func make() -> Self { Self() }`   | self внутри static — запрещено                       |

### 2. Самое важное правило 2026 года (Swift 6 strict concurrency)

С Swift 5.5–6.0 ввели **строгую конкурентность** и запрет неявного `self` в замыканиях.

```swift
// Ошибка в Swift 6 без capture list
someAsyncFunction {
    print(name)          // Error: Reference to captured var 'name' in concurrently-executing code
}
```

Правильный способ (2026 стандарт):

```swift
someAsyncFunction { [weak self] in
    guard let self else { return }
    print(self.name)
}

// или [self] — сильный захват (если уверены, что нет retain cycle)
someAsyncFunction { [self] in
    print(self.name)
}
```

**Коротко**:
- В замыканиях **всегда** явно захватывай `self` с помощью `[weak self]`, `[unowned self]` или `[self]`
- `[weak self]` + `guard let self` — самый безопасный и рекомендуемый паттерн

### 3. Полный разбор случаев (с примерами)

#### Случай 1. Конфликт имён (самый частый)

```swift
class Person {
    var name: String
    
    init(name: String) {
        self.name = name     // ← self обязательно!
        // name = name       // Ошибка: parameter 'name' used before being initialized
    }
    
    func rename(to name: String) {
        self.name = name     // ← снова обязательно
    }
}
```

#### Случай 2. Замыкания и capture list (самое важное в 2026)

```swift
class Counter {
    var count = 0
    
    func startTimer() {
        Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] _ in
            guard let self else { return }
            self.count += 1
            print("Count: \(self.count)")
        }
    }
}
```

Без `[weak self]` → [[retain cycle]] → утечка памяти  
Без `guard let self` → компилятор в Swift 6 выдаст ошибку

#### Случай 3. self в замыканиях без захвата (редко, но бывает)

```swift
lazy var lazyProperty: String = {
    return self.description   // ← self обязательно
}()
```

#### Случай 4. self как аргумент функции (передача экземпляра)

```swift
networkManager.fetchUser(self)  // ← self передаётся явно
```

#### Случай 5. self в extension (особенно с протоколами)

```swift
extension UIViewController: SomeProtocol {
    func doSomething() {
        self.title = "Hello"   // ← self можно опустить, но можно и оставить
    }
}
```

### 4. Когда self можно (и нужно) опускать

Swift позволяет **опускать** `self` почти везде, кроме:

- внутри замыканий (capture list)
- когда есть конфликт имён (параметр и свойство)
- в некоторых generic и associatedtype контекстах

Примеры, где self опускается (и это нормально):

```swift
func updateUI() {
    title = "Profile"           // self.title
    view.backgroundColor = .white // self.view
    label.text = "Hello"        // self.label.text
}
```

### 5. Короткая шпаргалка 2026

| Ситуация                           | Писать self?    | Пример правильного кода                  |
| ---------------------------------- | --------------- | ---------------------------------------- |
| Конфликт имён (init, метод)        | **Да**          | `self.name = name`                       |
| Внутри замыкания                   | **Да**          | `{ [weak self] in self?.doSomething() }` |
| Обычный метод, нет конфликта       | **Нет**         | `title = "Home"`                         |
| Передача экземпляра в функцию      | **Да**          | `delegate?.userDidLogin(self)`           |
| В [[extension]] протокола          | **Нет** (можно) | `title = "Settings"`                     |
| В статических методах / enum cases | **Нет**         | `static func make() -> Self { Self() }`  |

**Короткий девиз 2026**:
> `self` — это **не просто "я"**, это **явное указание на экземпляр**, чтобы избежать неоднозначности и retain cycle.  
> В 2026 году правило простое:  
> — конфликт имён или замыкание → пиши `self`  
> — в остальном → можно (и лучше) опускать для чистоты кода
