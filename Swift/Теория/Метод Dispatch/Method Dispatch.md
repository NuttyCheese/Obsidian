#swift #dispatch #method-dispatch #performance #static-dispatch #dynamic-dispatch #optimization

---
### Определение
**Method Dispatch (диспетчеризация методов)** — это механизм, определяющий, **какая именно реализация метода** будет вызвана при обращении к нему через переменную или ссылку . В Swift существует четыре основных вида диспетчеризации: **Direct (Static)**, **Table (vtable)**, **Witness Table** и **Message** .

Выбор вида диспетчеризации влияет на производительность, гибкость и возможности переопределения методов. Понимание этих механизмов необходимо для написания оптимального кода, особенно в горячих путях (циклы, рендеринг, аудиообработка) .

---

### Четыре вида Method Dispatch в Swift

| Вид диспетчеризации                                           | Время определения | Скорость | Полиморфизм | Переопределение | Инлайнинг     | Использование                                            |
| ------------------------------------------------------------- | ----------------- | -------- | ----------- | --------------- | ------------- | -------------------------------------------------------- |
| **[[Direct Dispatch\|Direct]] / [[Static Dispatch\|Static]]** | Компиляция        | ★★★★★    | Нет         | Нет             | Да            | [[struct]], [[enum]], [[final]], [[static]], [[private]] |
| **[[Table Dispatch\|Table]] (vtable)**                        | Выполнение        | ★★★★☆    | Да          | Да              | Редко         | Обычные методы классов                                   |
| **[[Witness Table]]**                                         | Выполнение        | ★★★★☆    | Да          | Да              | Редко         | Протоколы (existential)                                  |
| **[[Message Dispatch\|Message]]**                             | Выполнение        | ★★☆☆☆    | Да          | Да              | Почти никогда | [[@objc]], [[dynamic]], [[NSObject]]                     |

**Примерные цифры производительности:**
- **Direct Dispatch:** ~1–2 нс
- **Table Dispatch:** ~3–5 нс
- **Witness Table:** ~3–5 нс
- **Message Dispatch:** ~10–20 нс

---

### 1. Direct / Static Dispatch (прямая/статическая)

**Определение:** Компилятор на этапе компиляции точно знает, какая реализация метода будет вызвана, и заменяет вызов прямым переходом к коду метода .

**Когда используется:**
- Структуры ([[struct]]) и перечисления ([[enum]])
- `final` классы и `final` методы
- `static` методы
- `private` и `fileprivate` методы
- Методы в `extension` (даже для классов)
- [[Generic]] функции

```swift
struct Point {
    var x: Int
    var y: Int
    
    // Direct Dispatch
    func distance() -> Double {
        return sqrt(Double(x * x + y * y))
    }
}

final class Calculator {
    // Direct Dispatch
    func add(_ a: Int, _ b: Int) -> Int {
        return a + b
    }
}

let point = Point(x: 3, y: 4)
point.distance()  // Direct

let calc = Calculator()
calc.add(5, 3)    // Direct
```

**Преимущества:**
- Максимальная скорость (~1–2 нс)
- Возможность инлайнинга (компилятор может встроить код метода)
- Нет [[overhead]] на динамический поиск

**Недостатки:**
- Нет полиморфизма
- Нельзя переопределить в подклассах

---

### 2. Table Dispatch (vtable)

**Определение:** Компилятор создает таблицу виртуальных методов (vtable) для каждого класса. Вызов выполняется через поиск в этой таблице во время выполнения .

**Когда используется:**
- Обычные методы классов (не `final`, не `private`)
- Методы, которые могут быть переопределены

```swift
class Animal {
    // Table Dispatch
    func makeSound() {
        print("Generic sound")
    }
}

class Dog: Animal {
    // Table Dispatch (переопределение)
    override func makeSound() {
        print("Woof!")
    }
}

let animal: Animal = Dog()
animal.makeSound()  // Table Dispatch → "Woof!"
```

**Преимущества:**
- Полиморфизм (можно переопределять)
- Хорошая скорость (~3–5 нс)
- Не требует [[Objective-C]] [[Runtime]]

**Недостатки:**
- Медленнее Direct Dispatch
- Редко инлайнится

---

### 3. Witness Table Dispatch

**Определение:** При использовании протоколов через [[existential type]] ([[any Protocol]]) вызовы выполняются через witness table — таблицу указателей на реализации требований протокола .

**Когда используется:**
- Existential types (`any Protocol`)
- Параметры типа `any Protocol`
- Коллекции `[any Protocol]`

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

**Преимущества:**
- Полиморфизм для протоколов
- Хорошая скорость (~3–5 нс)
- Не требует наследования

**Недостатки:**
- Overhead памяти (~40–48 байт на existential)
- Медленнее Direct Dispatch

---

### 4. Message Dispatch

**Определение:** Вызов выполняется через Objective-C runtime (`objc_msgSend`). Это самый гибкий, но и самый медленный вид диспетчеризации .

**Когда используется:**
- Методы, помеченные `@objc dynamic`
- Наследование от `NSObject`
- [[KVO]], Method [[Swizzling]]
- [[UIKit]] методы (переопределения)

```swift
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

**Преимущества:**
- Динамическая замена методов (Method Swizzling)
- Поддержка KVO
- Совместимость с Objective-C

**Недостатки:**
- Самая низкая скорость (~10–20 нс)
- Почти никогда не инлайнится
- Требует Objective-C runtime

---

### Сравнительная таблица всех видов

| Характеристика | Direct | Table | Witness | Message |
|----------------|--------|-------|---------|---------|
| **Время определения** | Компиляция | Выполнение | Выполнение | Выполнение |
| **Скорость (нс)** | ~1–2 | ~3–5 | ~3–5 | ~10–20 |
| **Полиморфизм** | Нет | Да | Да | Да |
| **Переопределение** | Нет | Да | Да | Да |
| **Инлайнинг** | Да | Редко | Редко | Почти никогда |
| **Динамическая замена** | Нет | Нет | Нет | Да (Swizzling) |
| **KVO** | Нет | Нет | Нет | Да |
| **Где используется** | struct, final | классы | протоколы | @objc, NSObject |

---

### Правила определения вида диспетчеризации

| Конструкция | Вид диспетчеризации |
|-------------|---------------------|
| `struct` / `enum` | Direct |
| `final class` | Direct |
| `final` метод в классе | Direct |
| `static` метод | Direct |
| `private` / `fileprivate` метод | Direct |
| Метод в `extension` (класса) | Direct |
| Generic функция | Direct |
| Обычный метод класса | Table |
| `open` метод | Table |
| `@objc` метод (без `dynamic`) | Table (но доступен из Obj-C) |
| `@objc dynamic` метод | Message |
| Протокол (existential `any`) | Witness |
| Наследник `NSObject` (обычный метод) | Table |
| Переопределение метода `UIView` | Message |

---

### Производительность: пример измерения

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

### Оптимизации и best practices

#### 1. **Используйте `final` для классов, которые не наследуются**

```swift
// ✅ Хорошо — Direct Dispatch
final class FastService {
    func process() { }
}

// ❌ Медленнее — Table Dispatch
class SlowService {
    func process() { }
}
```

#### 2. **Помечайте методы `final` в классах**

```swift
class Service {
    // Direct Dispatch
    final func criticalPath() { }
    
    // Table Dispatch (для полиморфизма)
    func optionalOverride() { }
}
```

#### 3. **Используйте `struct` вместо `class` если не нужна ссылочная семантика**

```swift
// ✅ Direct Dispatch
struct Point {
    var x, y: Int
    func distance() { }
}

// ❌ Table Dispatch (если не нужен полиморфизм)
class PointClass {
    var x, y: Int
    func distance() { }
}
```

#### 4. **Используйте generics вместо existential**

```swift
// ❌ Existential — Witness Table
func draw(_ shape: any Drawable) {
    shape.draw()
}

// ✅ Generic — Direct Dispatch
func draw<T: Drawable>(_ shape: T) {
    shape.draw()
}
```

#### 5. **Избегайте `@objc dynamic` в горячих путях**

```swift
// ❌ Медленно — Message Dispatch
@objc dynamic var value: Int = 0

// ✅ Быстро — Table Dispatch
var value: Int = 0
```

---

### Сравнение видов диспетчеризации (визуализация)

```mermaid
flowchart TD
    subgraph "Direct / Static"
        A[Вызов] --> B[Прямой переход]
        B --> C[Выполнение]
    end
    
    subgraph "Table (vtable)"
        D[Вызов] --> E[Чтение vtable]
        E --> F[Поиск адреса]
        F --> G[Косвенный jump]
        G --> H[Выполнение]
    end
    
    subgraph "Witness Table"
        I[Вызов] --> J[Чтение existential container]
        J --> K[Поиск в witness table]
        K --> L[Косвенный jump]
        L --> M[Выполнение]
    end
    
    subgraph "Message"
        N[Вызов] --> O[objc_msgSend]
        O --> P[Поиск в runtime]
        P --> Q[Косвенный jump]
        Q --> R[Выполнение]
    end
```

---

### Короткое правило 2026

> **Method Dispatch** определяет, как вызываются методы в Swift.  
> **Direct** — самый быстрый, **Message** — самый гибкий, но медленный.  
> Используйте **`final`**, **`struct`**, **`static`**, **`private`** для скорости.  
> Используйте **`@objc dynamic`** только когда нужны KVO или Method Swizzling.  
> Для протоколов предпочитайте **generics** existential.

### Итог

**Method Dispatch** — фундаментальный механизм Swift, влияющий на производительность и гибкость кода:

1.  **Direct / Static:** Самый быстрый (~1–2 нс), нет полиморфизма.
2.  **Table (vtable):** Быстрый (~3–5 нс), полиморфизм для классов.
3.  **Witness Table:** Быстрый (~3–5 нс), полиморфизм для протоколов.
4.  **Message:** Медленный (~10–20 нс), максимальная гибкость (KVO, Swizzling).

**Выбор вида диспетчеризации — это баланс между производительностью и гибкостью.** Для горячих путей выбирайте Direct Dispatch (struct, final, static). Для полиморфизма используйте Table или Witness Dispatch. Message Dispatch используйте только когда действительно нужна динамическая замена методов или совместимость с Objective-C .