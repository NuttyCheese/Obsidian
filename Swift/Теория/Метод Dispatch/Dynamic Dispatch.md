#swift #dispatch #dynamic-dispatch #polymorphism #performance #table-dispatch #witness-table #message-dispatch

---
### Определение
**Dynamic Dispatch (динамическая диспетчеризация)** — это механизм вызова методов, при котором **реализация метода определяется во время выполнения** ([[Runtime]]), а не на этапе компиляции . Это позволяет реализовать полиморфизм — возможность вызывать разные реализации одного и того же метода в зависимости от фактического типа объекта .

В [[Swift]] существует три вида динамической диспетчеризации: **[[Table Dispatch]] (vtable)** для классов, **[[Witness Table]] Dispatch** для протоколов и **[[Message Dispatch]]** для [[Objective-C]] совместимости .

### Зачем это знать iOS-разработчику?
1.  **Полиморфизм:** Динамическая диспетчеризация — основа полиморфизма в ООП .
2.  **Производительность:** Динамические вызовы медленнее статических, важно знать где они происходят .
3.  **Оптимизация:** Использование `final` и `static` может превратить динамический вызов в статический .
4.  **Понимание протоколов:** Witness table — ключевой механизм для протоколов .
5.  **Objective-C совместимость:** Message dispatch необходим для [[KVO]], [[Swizzling]], [[UIKit]] .

---

### Динамическая vs Статическая диспетчеризация

| Характеристика | Статическая (Direct) | Динамическая (Table/Witness/Message) |
|----------------|----------------------|-------------------------------------|
| **Время определения** | Компиляция | Выполнение |
| **Скорость** | ★★★★★ (~1–2 нс) | ★★★☆☆ (~3–20 нс) |
| **Полиморфизм** | Нет | Да |
| **Переопределение** | Нет | Да |
| **Инлайнинг** | Да | Редко |
| **Гибкость** | Низкая | Высокая |

---

### Три вида Dynamic Dispatch в Swift

| Вид | Скорость | Использование | Пример |
|-----|----------|---------------|--------|
| **Table (vtable)** | ★★★★☆ (~3–5 нс) | Классы | `class Animal { func sound() }` |
| **Witness Table** | ★★★★☆ (~3–5 нс) | Протоколы (existential) | `any Drawable` |
| **Message** | ★★☆☆☆ (~10–20 нс) | @objc, dynamic, NSObject | `@objc dynamic func method()` |

---

### 1. Table Dispatch (vtable)

**Определение:** Компилятор создает таблицу виртуальных методов (vtable) для каждого класса. Вызов выполняется через поиск в этой таблице во время выполнения .

```swift
class Animal {
    // Table Dispatch
    func makeSound() {
        print("Generic sound")
    }
}

class Dog: Animal {
    // Table Dispatch — переопределение
    override func makeSound() {
        print("Woof!")
    }
}

class Cat: Animal {
    override func makeSound() {
        print("Meow!")
    }
}

let animals: [Animal] = [Dog(), Cat(), Animal()]
for animal in animals {
    animal.makeSound()  // Table Dispatch → разные звуки
}
```

**Как работает:**
- Каждый класс имеет vtable — массив указателей на методы.
- Объект хранит указатель на vtable своего класса.
- Вызов метода: `object->vtable[index]()`.

---

### 2. Witness Table Dispatch

**Определение:** При использовании протоколов через existential type (`any Protocol`) вызовы выполняются через witness table — таблицу указателей на реализации требований протокола .

```swift
protocol Drawable {
    func draw()
}

struct Circle: Drawable {
    func draw() { print("○") }
}

struct Square: Drawable {
    func draw() { print("□") }
}

let shapes: [any Drawable] = [Circle(), Square()]

for shape in shapes {
    shape.draw()  // Witness Table Dispatch
}
```

**Как работает:**
- Existential container хранит value buffer, vwt и pwt.
- Witness table содержит указатели на методы протокола.

---

### 3. Message Dispatch

**Определение:** Вызов выполняется через Objective-C runtime (`objc_msgSend`). Самый гибкий, но и самый медленный вид .

```swift
import Foundation

class MyClass: NSObject {
    // Message Dispatch
    @objc dynamic func messageMethod() {
        print("Message dispatch")
    }
    
    // Table Dispatch
    func tableMethod() {
        print("Table dispatch")
    }
}

let obj = MyClass()
obj.messageMethod()  // objc_msgSend
obj.tableMethod()    // vtable
```

**Как работает:**
- Компилятор генерирует вызов `objc_msgSend(receiver, selector)`.
- Runtime ищет реализацию в иерархии классов.
- Поддерживает Method Swizzling и KVO.

---

### Сравнение видов Dynamic Dispatch

```mermaid
flowchart TD
    subgraph "Table (vtable)"
        A[Вызов метода] --> B[Чтение vtable из объекта]
        B --> C[Поиск адреса по индексу]
        C --> D[Косвенный jump]
        D --> E[Выполнение]
    end
    
    subgraph "Witness Table"
        F[Вызов метода] --> G[Чтение existential container]
        G --> H[Поиск в witness table]
        H --> I[Косвенный jump]
        I --> J[Выполнение]
    end
    
    subgraph "Message"
        K[Вызов метода] --> L[objc_msgSend]
        L --> M[Поиск в runtime cache]
        M --> N[Косвенный jump]
        N --> O[Выполнение]
    end
```

---

### Производительность: сравнение

```swift
import Darwin

protocol TestProtocol {
    func witnessMethod()
}

struct TestStruct: TestProtocol {
    func witnessMethod() { }
}

class TestClass {
    func tableMethod() { }
    final func directMethod() { }
    @objc dynamic func messageMethod() { }
}

func measure(_ name: String, iterations: Int, _ block: () -> Void) {
    let start = mach_absolute_time()
    for _ in 0..<iterations {
        block()
    }
    let end = mach_absolute_time()
    
    var info = mach_timebase_info()
    mach_timebase_info(&info)
    let elapsed = (end - start) * UInt64(info.numer) / UInt64(info.denom)
    let avg = Double(elapsed) / Double(iterations)
    print("\(name): \(String(format: "%.2f", avg)) нс")
}

let testClass = TestClass()
let testStruct = TestStruct()
let existential: any TestProtocol = testStruct

measure("Direct (struct)", iterations: 10_000_000) {
    testStruct.witnessMethod()
}

measure("Table (class)", iterations: 10_000_000) {
    testClass.tableMethod()
}

measure("Witness (existential)", iterations: 10_000_000) {
    existential.witnessMethod()
}

measure("Message (@objc dynamic)", iterations: 10_000_000) {
    testClass.messageMethod()
}

// Примерный результат:
// Direct (struct): 1.2 нс
// Table (class): 3.5 нс
// Witness (existential): 3.8 нс
// Message (@objc dynamic): 15.2 нс
```

---

### Когда используется каждый вид Dynamic Dispatch

| Вид | Использование | Пример |
|-----|---------------|--------|
| **Table** | Обычные методы классов | `class Animal { func sound() }` |
| **Table** | Переопределенные методы | `override func sound()` |
| **Witness** | Existential types | `let shape: any Drawable = Circle()` |
| **Witness** | Коллекции протоколов | `[any Drawable]` |
| **Message** | `@objc dynamic` методы | `@objc dynamic func update()` |
| **Message** | NSObject наследники (некоторые методы) | `override func touchesBegan()` |
| **Message** | KVO свойства | `@objc dynamic var value` |

---

### Оптимизации: превращение Dynamic в Static

#### 1. **Используйте final**

```swift
// Dynamic → Static
final class FastClass {
    func method() { }  // Static Dispatch
}

class NormalClass {
    final func method() { }  // Static Dispatch
}
```

#### 2. **Используйте private / fileprivate**

```swift
class MyClass {
    private func helper() { }  // Static Dispatch
}
```

#### 3. **Используйте static**

```swift
class Math {
    static func square(_ x: Int) -> Int {  // Static Dispatch
        return x * x
    }
}
```

#### 4. **Используйте [[generic]]s вместо existential**

```swift
// Dynamic (Witness)
func draw(_ shape: any Drawable) {
    shape.draw()
}

// Static (Direct)
func draw<T: Drawable>(_ shape: T) {
    shape.draw()
}
```

---

### Когда Dynamic Dispatch необходим

| Сценарий | Почему необходим |
|----------|------------------|
| **Полиморфизм классов** | Разные реализации в подклассах |
| **Протоколы с разными типами** | Коллекции разных типов, соответствующих протоколу |
| **KVO** | Требует Message Dispatch |
| **Method Swizzling** | Требует Message Dispatch |
| **UIKit наследование** | Многие методы UIKit используют Message Dispatch |

---

### Лучшие практики

#### 1. **По умолчанию используйте static, где возможно**

```swift
// ✅ Быстро
final class Service {
    func process() { }
}

// ❌ Медленнее, если полиморфизм не нужен
class Service {
    func process() { }
}
```

#### 2. **Для протоколов используйте generics**

```swift
// ✅ Static dispatch
func process<T: Drawable>(_ shape: T) {
    shape.draw()
}

// ❌ Witness table overhead
func process(_ shape: any Drawable) {
    shape.draw()
}
```

#### 3. **Ограничьте использование @objc dynamic**

```swift
// ❌ Медленно, если не нужны KVO/Swizzling
@objc dynamic var value: Int = 0

// ✅ Быстро
var value: Int = 0
```

#### 4. **Используйте `some` вместо `any` для возвращаемых значений**

```swift
// ❌ Existential — witness table
func makeDrawable() -> any Drawable {
    return Circle()
}

// ✅ Opaque type — static
func makeDrawable() -> some Drawable {
    return Circle()
}
```

---

### Короткое правило 2026

> **Dynamic Dispatch** — основа полиморфизма в Swift, но плата за гибкость — производительность.  
> Используйте `final`, `static`, `private` и **generics**, чтобы превратить динамические вызовы в статические там, где это возможно.  
> Message Dispatch используйте только для KVO, Swizzling и совместимости с Objective-C.

### Итог

**Dynamic Dispatch** — мощный механизм, обеспечивающий полиморфизм в Swift:

1.  **Table (vtable):** Для классов (~3–5 нс) — быстрый, полиморфизм.
2.  **Witness Table:** Для протоколов (~3–5 нс) — быстрый, полиморфизм.
3.  **Message:** Для Objective-C (~10–20 нс) — медленный, максимальная гибкость.

**Выбор между статической и динамической диспетчеризацией — это баланс между производительностью и гибкостью.** В горячих путях предпочитайте статическую диспетчеризацию (struct, final, generics). Динамическую используйте, когда нужен полиморфизм или динамические возможности Objective-C .