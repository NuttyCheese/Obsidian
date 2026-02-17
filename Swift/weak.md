**`weak`** — это модификатор ссылки на объект, который:

- **Не увеличивает счетчик ссылок ([[ARC]])**
    
- Становится **[[nil]] автоматически**, когда объект, на который она ссылается, освобождается
    
- Всегда должен быть **[[Optional]]** (`?`), потому что объект может исчезнуть в любой момент
    

> Проще говоря: `weak` = «ссылка на объект, которая не удерживает его и может стать nil».

---

## 2. Основные термины

| Термин                                 | Описание                                                                           |
| -------------------------------------- | ---------------------------------------------------------------------------------- |
| **ARC (Automatic Reference Counting)** | Автоматический подсчет ссылок на объекты в Swift                                   |
| **Strong Reference**                   | Сильная ссылка, увеличивает счетчик ARC                                            |
| **Weak Reference (`weak`)**            | Слабая ссылка, не увеличивает счетчик ARC, становится nil при освобождении объекта |
| **Unowned Reference (`unowned`)**      | Слабая ссылка, не optional, crash если объект освобождён                           |
| **[[retain cycle]]**                  | Цикл сильных ссылок между объектами, приводящий к утечке памяти                    |

---

## 3. Основной синтаксис

```swift
class Person {
    var name: String
    init(name: String) { self.name = name }
}

class Apartment {
    var number: Int
    weak var tenant: Person?  // weak ссылка
    init(number: Int) {
        self.number = number
    }
}
```

- `weak var tenant: Person?` — объект `Apartment` ссылается на `Person`, **не удерживая его**
    
- Ссылка Optional, может стать `nil` при освобождении `Person`
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простейший weak

```swift
class Owner {
    let name: String
    init(name: String) { self.name = name }
}

class Pet {
    weak var owner: Owner?  // weak ссылка
    init(owner: Owner) { self.owner = owner }
}

var alice: Owner? = Owner(name: "Alice")
var pet: Pet? = Pet(owner: alice!)
print(pet?.owner?.name) // Alice

alice = nil
print(pet?.owner?.name) // nil
```

- Ссылка `owner` не удерживает объект `alice`, становится nil при его освобождении
    

---

### Пример 2. weak vs [[unowned]]

```swift
class A {
    var b: B?
}

class B {
    weak var a: A?  // может быть nil
    init(a: A) { self.a = a }
}

var objectA: A? = A()
var objectB: B? = B(a: objectA!)
objectA?.b = objectB

objectA = nil
print(objectB?.a) // nil, безопасно
```

- `weak` безопаснее, чем `unowned`, так как не крашит приложение
    

---

### Пример 3. Замыкания с weak

```swift
class ViewController {
    var closure: (() -> Void)?

    func setupClosure() {
        closure = { [weak self] in
            guard let self = self else { return }
            print("ViewController is \(self)")
        }
    }
}
```

- Используем `[weak self]` чтобы избежать retain cycle в замыкании
    
- Всегда проверяем, что [[self]] ещё существует
    

---

### Пример 4. Слабая ссылка на делегата

```swift
protocol WorkerDelegate: AnyObject {
    func didFinishWork()
}

class Worker {
    weak var delegate: WorkerDelegate?
    func doWork() {
        // Работа
        delegate?.didFinishWork()
    }
}
```

- Делегаты всегда weak, чтобы избежать retain cycle между объектом и делегатом
    

---

### Пример 5. Цикл ссылок с weak

```swift
class Parent {
    var child: Child?
}

class Child {
    weak var parent: Parent?  // слабая ссылка, чтобы избежать retain cycle
}

var parent: Parent? = Parent()
var child: Child? = Child()

parent?.child = child
child?.parent = parent

parent = nil
child = nil
```

- Без weak циклы ссылок могут приводить к утечкам памяти
    

---

## 5. Особенности `weak`

1. **Optional** (`?`) — потому что объект может исчезнуть
    
2. **Не увеличивает счетчик ARC**, предотвращает retain cycle
    
3. Используется для:
    
    - Ссылок на делегатов
        
    - Ссылок между объектами с разной продолжительностью жизни
        
    - Замыканий
        
4. Безопасен: ссылка автоматически становится `nil`
    

---

## 6. Итог

- **weak** = слабая ссылка на объект, которая **может стать nil**
    
- Используется для:
    
    - Предотвращения retain cycle
        
    - Делегатов
        
    - Замыканий
        
- Отличие от `unowned`: **weak optional и безопасен**, `unowned` не optional и crash при освобождении
    

---
