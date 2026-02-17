**`unowned`** — это модификатор ссылки на объект, который:

- **Не увеличивает счетчик ссылок ([[ARC]])**
    
- Предполагает, что объект, на который ссылаются, **не будет освобождён** до того, как ссылка будет использоваться
    
- Если объект уже освобождён, попытка обращения к unowned-ссылке **приведет к крашу приложения**
    

> Проще говоря: `unowned` = «ссылка на объект, который точно будет жить столько же, сколько и эта ссылка».

---

## 2. Основные термины

| Термин                                 | Описание                                                                      |
| -------------------------------------- | ----------------------------------------------------------------------------- |
| **ARC (Automatic Reference Counting)** | Автоматический подсчет ссылок на объекты в Swift                              |
| **Strong Reference**                   | Сильная ссылка, увеличивает счетчик ARC                                       |
| **Weak Reference ([[weak]])**          | Слабая ссылка, не увеличивает счетчик ARC и может стать [[nil]]               |
| **Unowned Reference (`unowned`)**      | Слабая ссылка, не увеличивает счетчик ARC, **не optional**, не может быть nil |
| **[[retain cycle]]**                  | Цикл сильных ссылок между объектами, приводящий к утечке памяти               |

---

## 3. Основной синтаксис

```swift
class Person {
    var name: String
    init(name: String) {
        self.name = name
    }
}

class Apartment {
    var number: Int
    unowned var tenant: Person  // unowned ссылка
    init(number: Int, tenant: Person) {
        self.number = number
        self.tenant = tenant
    }
}
```

- `unowned var tenant: Person` — объект `Apartment` ссылается на `Person`, **не увеличивая счетчик ARC**
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простейший unowned

```swift
class Owner {
    let name: String
    init(name: String) { self.name = name }
}

class Pet {
    let name: String
    unowned let owner: Owner
    init(name: String, owner: Owner) {
        self.name = name
        self.owner = owner
    }
}

let owner = Owner(name: "Alice")
let pet = Pet(name: "Fluffy", owner: owner)
print(pet.owner.name) // Alice
```

- Ссылка `owner` не увеличивает счетчик ARC
    

---

### Пример 2. Unowned vs Weak

```swift
class A {
    var b: B?
}

class B {
    unowned var a: A  // Не optional, предполагается, что A существует
    init(a: A) { self.a = a }
}

var objectA: A? = A()
var objectB: B? = B(a: objectA!)
objectA?.b = objectB

objectA = nil
// objectB?.a -> crash, потому что объект A уже освобождён
```

- `unowned` безопасно, если владеющий объект живёт дольше
    

---

### Пример 3. Замыкания с unowned

```swift
class ViewController {
    var closure: (() -> Void)?

    func setupClosure() {
        closure = { [unowned self] in
            print("ViewController is \(self)")
        }
    }
}
```

- Используем `[unowned self]` чтобы избежать retain cycle в замыкании
    

---

### Пример 4. Unowned с [[Optional]] контекстом

```swift
class Customer {
    let name: String
    var card: CreditCard?
    init(name: String) { self.name = name }
}

class CreditCard {
    let number: UInt
    unowned let customer: Customer
    init(number: UInt, customer: Customer) {
        self.number = number
        self.customer = customer
    }
}

let john = Customer(name: "John Appleseed")
john.card = CreditCard(number: 1234_5678_9012_3456, customer: john)
```

- `CreditCard` ссылается на `Customer`, не создавая retain cycle
    

---

### Пример 5. Замыкание в сложной структуре

```swift
class Node {
    var value: Int
    var next: Node?
    init(value: Int) { self.value = value }
    lazy var printValue: () -> Void = { [unowned self] in
        print("Node value is \(self.value)")
    }
}

let node = Node(value: 10)
node.printValue() // Node value is 10
```

- Замыкание с `[unowned self]` безопасно, если Node живёт дольше замыкания
    

---

## 5. Особенности `unowned`

1. **Не optional**, всегда предполагается, что объект существует
    
2. **Не увеличивает счетчик ARC**, предотвращает retain cycle
    
3. Используется для **объектов с более долгим временем жизни**, чем текущий объект
    
4. Ошибки при доступе к уже освобождённому объекту приводят к **crash**
    
5. Отличие от `weak`: `weak` optional, может стать nil; `unowned` не optional, crash если объект освобождён
    

---

## 6. Итог

- **unowned** = слабая ссылка без увеличения ARC, предполагает, что объект живёт дольше
    
- Используется для:
    
    - Связей между объектами с разной продолжительностью жизни
        
    - Замыканий для предотвращения retain cycle
        
- Отличие от `weak`: **не optional, не может быть nil**, crash при освобождении объекта
    

---
