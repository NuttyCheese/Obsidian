#swift #properties #set #computed-properties #property-observers #access-control

---
### Определение
**`set`** — это блок кода, определяющий, как присваивается новое значение свойству. В [[Swift]] `set` используется в двух основных контекстах:

1.  **Вычисляемые свойства (computed properties):** `set` определяет логику присвоения значения.
2.  **Наблюдатели свойств (property observers):** `set` является частью наблюдателей [[willSet]] и [[didSet]], которые реагируют на изменение значения.

### Зачем это знать iOS-разработчику?
1.  **Контроль присваивания:** Можно валидировать, преобразовывать или реагировать на новые значения.
2.  **Инкапсуляция:** Скрывать внутреннее представление данных, предоставляя удобный интерфейс.
3.  **Реактивность:** `didSet` позволяет выполнять код при изменении свойства (обновление UI, логирование).
4.  **Вычисляемые свойства:** Создавать свойства, которые не хранятся напрямую, а вычисляются на основе других свойств.

---

### 1. `set` в вычисляемых свойствах

Вычисляемые свойства не хранят значение, а вычисляют его при получении ([[getter|get]]) и могут устанавливать значение через `set`.

#### Синтаксис

```swift
var propertyName: Type {
    get {
        // код для получения значения
        return someValue
    }
    set(newValue) {
        // код для установки значения
        // newValue — автоматически предоставляемое имя нового значения
    }
}
```

- Если не указать имя параметра в `set`, доступно имя по умолчанию — **`newValue`**.
- `set` может использоваться без `get` только если свойство только для записи (редко).

#### Пример: Прямоугольник с площадью

```swift
struct Rectangle {
    var width: Double
    var height: Double
    
    // Вычисляемое свойство area
    var area: Double {
        get {
            return width * height
        }
        set(newArea) {
            // При установке площади сохраняем пропорции
            let ratio = width / height
            height = sqrt(newArea / ratio)
            width = height * ratio
        }
    }
}

var rect = Rectangle(width: 4, height: 3)
print(rect.area)  // 12.0 (get)

rect.area = 48     // set
print(rect.width)  // 8.0
print(rect.height) // 6.0
```

#### Пример: с использованием `newValue`

```swift
struct Temperature {
    var celsius: Double
    
    var fahrenheit: Double {
        get {
            return celsius * 9 / 5 + 32
        }
        set {
            // Используем newValue по умолчанию
            celsius = (newValue - 32) * 5 / 9
        }
    }
}

var temp = Temperature(celsius: 25)
print(temp.fahrenheit)  // 77.0

temp.fahrenheit = 68
print(temp.celsius)     // 20.0
```

#### Пример: свойство только для записи (редко)

```swift
struct User {
    var name: String
    private var _age: Int = 0
    
    var age: Int {
        set {
            _age = max(0, newValue)  // только установка, без чтения
        }
    }
    
    func getAge() -> Int {
        return _age
    }
}

var user = User(name: "Alice")
user.age = 25
print(user.getAge())  // 25
```

---

### 2. `willSet` и `didSet` — наблюдатели свойств

Наблюдатели позволяют выполнять код до и после изменения значения свойства.

#### Синтаксис

```swift
var propertyName: Type = initialValue {
    willSet(newValue) {
        // Вызывается перед установкой нового значения
        // newValue — новое значение (имя можно изменить)
    }
    didSet(oldValue) {
        // Вызывается после установки нового значения
        // oldValue — старое значение (имя можно изменить)
    }
}
```

#### Пример: Логирование изменений

```swift
class BankAccount {
    var balance: Double = 0.0 {
        willSet {
            print("Баланс изменится с \(balance) на \(newValue)")
        }
        didSet {
            if oldValue > balance {
                print("Снято \(oldValue - balance)")
            } else if balance > oldValue {
                print("Зачислено \(balance - oldValue)")
            }
        }
    }
}

let account = BankAccount()
account.balance = 100
// Баланс изменится с 0.0 на 100.0
// Зачислено 100.0

account.balance = 75
// Баланс изменится с 100.0 на 75.0
// Снято 25.0
```

#### Пример: Валидация в `willSet`

```swift
struct Person {
    var name: String {
        willSet {
            guard !newValue.isEmpty else {
                fatalError("Имя не может быть пустым")
            }
        }
    }
}

var person = Person(name: "Alice")
// person.name = ""  // fatalError: Имя не может быть пустым
```

#### Пример: Обновление UI в `didSet` ([[iOS]])

```swift
import UIKit

class ProfileViewController: UIViewController {
    @IBOutlet weak var nameLabel: UILabel!
    
    var userName: String = "" {
        didSet {
            nameLabel.text = userName
        }
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        userName = "John Doe"  // Автоматически обновит label
    }
}
```

---

### 3. Комбинирование `set` с наблюдателями

Для хранимых свойств можно использовать только наблюдатели. Для вычисляемых свойств — только `set` (и `get`).

```swift
class ViewModel {
    // Хранимое свойство → наблюдатели
    var name: String = "" {
        didSet {
            print("Name changed from \(oldValue) to \(name)")
        }
    }
    
    // Вычисляемое свойство → get/set
    var formattedName: String {
        get {
            return name.uppercased()
        }
        set {
            name = newValue.lowercased()
        }
    }
}

let vm = ViewModel()
vm.name = "Alice"      // Name changed from  to Alice
vm.formattedName = "BOB"
print(vm.name)         // bob
print(vm.formattedName) // BOB
```

---

### 4. `set` с модификаторами доступа

Можно делать `get` публичным, а `set` приватным.

```swift
struct User {
    private(set) var id: Int  // set доступен только внутри структуры
    var name: String
    
    init(id: Int, name: String) {
        self.id = id
        self.name = name
    }
    
    mutating func updateId(newId: Int) {
        id = newId  // доступен внутри
    }
}

var user = User(id: 1, name: "Alice")
// user.id = 2  // Ошибка: 'id' setter is inaccessible
user.updateId(newId: 2)
print(user.id)  // 2 (get доступен)
```

---

### 5. `set` в расширениях (extensions)

В расширениях можно добавлять только вычисляемые свойства (с `get` и `set`), но нельзя добавлять хранимые свойства с наблюдателями.

```swift
extension String {
    var reversed: String {
        get {
            return String(self.reversed())
        }
        set {
            self = newValue.reversed()  // set принимает строку
        }
    }
}

var text = "Hello"
text.reversed = "olleH"
print(text)  // Hello (так как set записывает reversed)
```

---

### 6. `set` и управление памятью

Для классов можно использовать `weak` и `unowned` в замыканиях внутри `set`, чтобы избежать [[retain cycle]]s.

```swift
class DataManager {
    var onUpdate: (() -> Void)?
}

class ViewController {
    var dataManager = DataManager()
    var value: String = "" {
        didSet {
            // Без [weak self] может быть retain cycle
            dataManager.onUpdate = { [weak self] in
                self?.updateUI()
            }
        }
    }
    
    func updateUI() {
        print("UI updated")
    }
}
```

---

### Лучшие практики

#### 1. **Не делайте тяжелых операций в `set`**

```swift
// ❌ Плохо — сеттер выполняет тяжелую работу
var images: [UIImage] = [] {
    didSet {
        DispatchQueue.global().async {
            self.generateThumbnails()  // долго
        }
    }
}

// ✅ Хорошо — разделяйте логику
var images: [UIImage] = [] {
    didSet {
        scheduleThumbnailGeneration()
    }
}
```

#### 2. **Используйте `didSet` для обновления UI**

```swift
class UserProfileView {
    var user: User? {
        didSet {
            updateUI()
        }
    }
    
    private func updateUI() {
        nameLabel.text = user?.name
        avatarImageView.image = user?.avatar
    }
}
```

#### 3. **Валидируйте в `willSet`, а не в `didSet`**

```swift
var age: Int = 0 {
    willSet {
        guard newValue >= 0 else {
            fatalError("Age cannot be negative")
        }
    }
    didSet {
        print("Age changed from \(oldValue) to \(age)")
    }
}
```

#### 4. **Не используйте наблюдатели для свойств, которые не изменяются**

Лишние вызовы `didSet` могут снижать производительность.

---

### Сравнение: `set` vs `willSet`/`didSet`

| Характеристика | `set` (вычисляемые свойства) | `willSet`/`didSet` (хранимые свойства) |
|----------------|------------------------------|----------------------------------------|
| **Применение** | Вычисляемые свойства | Хранимые свойства |
| **Хранение значения** | Нет (вычисляется) | Да (значение хранится) |
| **Доступ к новому значению** | `newValue` (или параметр) | `newValue` в `willSet` |
| **Доступ к старому значению** | Нет | `oldValue` в `didSet` |
| **Предотвращение присваивания** | Можно (не сохранять) | Нельзя (значение уже установлено) |

---

### Короткое правило

> **`set`** определяет логику присваивания.  
> Для **вычисляемых свойств** — преобразует входящее значение.  
> Для **хранимых свойств** — используйте `willSet` (до) и `didSet` (после) для наблюдения.  
> Не делайте тяжелых операций в сеттерах.

### Итог

**`set`** в Swift:

1.  **Используется в вычисляемых свойствах** для установки значения.
2.  **Наблюдатели `willSet`/`didSet`** позволяют реагировать на изменения хранимых свойств.
3.  **Позволяет контролировать и валидировать** присваиваемые значения.
4.  **Инкапсулирует логику** преобразования данных.
5.  **Может быть приватным** (`private(set)`) для ограничения записи извне.

Понимание работы сеттеров помогает писать чистый, предсказуемый и безопасный код .