[[Swift]] сообщает, что **вы попытались обратиться к объекту через `unowned` ссылку, который уже был освобождён (deallocated)**.

- `unowned` — это **не-владельческая ссылка**, которая **не удерживает объект в памяти**.
    
- Отличие от [[weak]]:
    
    - `weak` — автоматически становится [[nil]], если объект уничтожен.
        
    - [[unowned]] — предполагает, что объект **существует всегда**, и если это не так → [[Runtime]] crash.
        
- Ошибка часто возникает при **retain cycle-breaking closures** и неправильном использовании `unowned`.
    

---

### Примеры кода/сценариев возникновения

**Пример 1: Замыкание с `unowned self`**

```swift
class Person {
    let name: String
    init(name: String) { self.name = name }
    
    func greet() {
        print("Hello, \(name)")
    }
    
    func scheduleGreeting() -> () -> Void {
        return { [unowned self] in
            self.greet() // ❌ если Person уже деаллоцирован, runtime crash
        }
    }
}

var p: Person? = Person(name: "Alice")
let greeting = p!.scheduleGreeting()
p = nil // Person деаллоцирован
greeting() // ❌ Attempted to read an unowned reference but object was deallocated
```

---

**Пример 2: `unowned` в структурах и классах**

```swift
class Manager {
    var employee: Employee?
}

class Employee {
    unowned let manager: Manager
    init(manager: Manager) {
        self.manager = manager
    }
}

var m: Manager? = Manager()
var e: Employee? = Employee(manager: m!)
m = nil
// ❌ попытка доступа к e.manager приведёт к runtime crash
```

---

### Как исправить

#### 1️⃣ Использовать `weak` вместо `unowned`

```swift
return { [weak self] in
    self?.greet() // ✅ безопасно, если self nil — замыкание не выполняется
}
```

---

#### 2️⃣ Гарантировать, что объект жив во время доступа

- `unowned` безопасен только если жизненный цикл объекта **дольше замыкания/ссылки**.
    
- Например, если объект создаётся вместе с замыканием и разрушается позже.
    

---

#### 3️⃣ Проверять жизненный цикл объектов при использовании `unowned`

- Не использовать `unowned` для объектов, которые могут быть деаллоцированы раньше.
    
- Использовать `weak` или явную проверку `guard let self = self else { return }`.
    

```swift
return { [weak self] in
    guard let self = self else { return }
    self.greet()
}
```

---

### Резюме

- Ошибка возникает, когда **unowned ссылка указывает на объект, который уже освобождён**.
    
- Исправляется через:
    
    1. Замена `unowned` на `weak`, если объект может быть деаллоцирован.
        
    2. Контроль жизненного цикла объектов, чтобы гарантировать существование объекта при использовании `unowned`.
        
- Позволяет избежать **runtime crash** и безопасно работать с замыканиями и ссылками на объекты.
    

---
