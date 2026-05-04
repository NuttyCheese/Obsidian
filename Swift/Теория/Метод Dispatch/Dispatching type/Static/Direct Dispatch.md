#swift #dispatch #direct-dispatch #static-dispatch #performance #optimization #swift6

---
### Определение
**Direct Dispatch (прямая диспетчеризация)** — это вид статической диспетчеризации, при котором компилятор на этапе компиляции точно определяет адрес вызываемого метода и заменяет вызов прямым переходом (jump или call) к этому адресу . Это самый быстрый и предсказуемый способ вызова функций в [[Swift]].

Direct Dispatch является частным случаем **[[Static Dispatch]]**, но термин часто используется как синоним. Ключевая особенность — компилятор **не генерирует таблицу виртуальных методов** (vtable) и **не использует динамический поиск** во время выполнения .

### Зачем это знать [[iOS]]-разработчику?
1.  **Максимальная производительность:** Direct Dispatch — самый быстрый вид диспетчеризации (~1–2 нс на вызов) .
2.  **Инлайнинг:** Компилятор может встроить код метода прямо в место вызова, устраняя overhead вызова полностью .
3.  **Оптимизация горячих путей:** Критичен для циклов, рендеринга, аудиообработки, ML-инференса .
4.  **Предсказуемость:** Нет динамического поиска, что упрощает отладку и анализ .
5.  **Swift 6 strict concurrency:** Статическая диспетчеризация упрощает проверку потокобезопасности .

---

### Direct Dispatch vs Другие виды диспетчеризации

| Вид диспетчеризации | Время определения | Скорость | Полиморфизм | Переопределение | Инлайнинг | Overhead |
|---------------------|-------------------|----------|-------------|-----------------|-----------|---------|
| **Direct / Static** | Компиляция | ★★★★★ | Нет | Нет | Да (часто) | Минимальный |
| **Table (vtable)** | Выполнение | ★★★★☆ | Да | Да | Редко | Средний |
| **Witness Table** | Выполнение | ★★★★☆ | Да | Да | Редко | Средний |
| **Message** | Выполнение | ★★☆☆☆ | Да | Да | Почти никогда | Высокий |

**Примерные цифры производительности:**
- **Direct Dispatch:** ~1–2 нс
- **[[Table Dispatch]]:** ~3–5 нс
- **[[Witness Table]]:** ~3–5 нс
- **[[Message Dispatch]]:** ~10–20 нс

---

### Когда используется Direct Dispatch

#### 1. **Структуры и перечисления**

```swift
struct Point {
    var x: Int
    var y: Int
    
    // Все методы struct — Direct Dispatch
    func distance() -> Double {
        return sqrt(Double(x * x + y * y))
    }
}

enum Status {
    case success, failure
    
    // Все методы enum — Direct Dispatch
    func isSuccess() -> Bool {
        return self == .success
    }
}

let point = Point(x: 3, y: 4)
point.distance()  // Direct вызов
```

#### 2. **Методы, помеченные [[final]]**

```swift
class Animal {
    // final — нельзя переопределить → Direct Dispatch
    final func breathe() {
        print("Breathing")
    }
    
    // Обычный метод → Table Dispatch
    func move() {
        print("Moving")
    }
}

class Dog: Animal {
    // Нельзя переопределить breathe()
    override func move() {  // Table Dispatch
        print("Running")
    }
}

let animal = Animal()
animal.breathe()  // Direct
animal.move()     // Table
```

#### 3. **Методы [[static]] и [[class]] [[final]]**

```swift
class Math {
    // static → Direct Dispatch
    static func square(_ x: Int) -> Int {
        return x * x
    }
    
    // class final → Direct Dispatch
    class final func cube(_ x: Int) -> Int {
        return x * x * x
    }
}

Math.square(5)   // Direct
Math.cube(3)     // Direct
```

#### 4. **Методы в [[extension]]s**

```swift
class Person {
    var name: String
    init(name: String) { self.name = name }
}

// Extension методы — Direct Dispatch
extension Person {
    func greet() {
        print("Hello, I'm \(name)")
    }
}

let person = Person(name: "Alice")
person.greet()  // Direct

class Student: Person {
    // ❌ Нельзя override extension метод
    // override func greet() { }  // Ошибка!
}
```

#### 5. **Приватные и [[fileprivate]] методы**

```swift
class Service {
    // private → Direct Dispatch (внутри файла)
    private func helper() {
        print("helper")
    }
    
    // fileprivate → Direct Dispatch (внутри файла)
    fileprivate func internalHelper() {
        print("internal")
    }
    
    func process() {
        helper()  // Direct
    }
}
```

#### 6. **[[Generic]] функции**

```swift
protocol Drawable {
    func draw()
}

struct Circle: Drawable {
    func draw() { print("○") }
}

// Generic функция → Direct Dispatch
func drawShape<T: Drawable>(_ shape: T) {
    shape.draw()  // Direct (тип известен на этапе компиляции)
}

let circle = Circle()
drawShape(circle)  // Direct вызов Circle.draw()
```

#### 7. **Opaque types ([[some]])**

```swift
func makeDrawable() -> some Drawable {
    return Circle()  // конкретный тип скрыт, но известен компилятору
}

let shape = makeDrawable()
shape.draw()  // Direct (Circle.draw())
```

---

### Как компилятор обрабатывает Direct Dispatch

```swift
// Исходный код
final class Calculator {
    func add(_ a: Int, _ b: Int) -> Int {
        return a + b
    }
}

let calc = Calculator()
let result = calc.add(5, 3)
```

**Упрощенный SIL (Swift Intermediate Language):**
```swift
// После компиляции — прямая ссылка на функцию
%1 = function_ref @Calculator.add(_:_:)
%2 = apply %1(%1, 5, 3)  // прямой вызов
```

**Возможный ассемблерный код (x86-64):**
```asm
mov eax, 5
mov ebx, 3
add eax, ebx      ; код метода встроен или вызван напрямую
```

---

### Direct Dispatch и инлайнинг

Компилятор может **инлайнить** методы, вызываемые через Direct Dispatch, полностью устраняя overhead вызова.

```swift
final class Math {
    func square(_ x: Int) -> Int {
        return x * x
    }
}

// В release сборке вызов может быть заменен на:
// let result = 5 * 5
let result = Math().square(5)  // инлайнится
```

**Сравнение с динамической диспетчеризацией:**
```swift
// Direct Dispatch — может инлайниться
final class FastMath {
    func add(_ a: Int, _ b: Int) -> Int { a + b }
}

// Table Dispatch — не может инлайниться
class SlowMath {
    func add(_ a: Int, _ b: Int) -> Int { a + b }
}
```

---

### Правила определения Direct Dispatch

| Конструкция                           | Вид диспетчеризации | Примечание              |
| ------------------------------------- | ------------------- | ----------------------- |
| **[[struct]] / [[enum]]**             | Direct              | Все методы              |
| **final class**                       | Direct              | Все методы              |
| **[[final]] метод в классе**          | Direct              | Для этого метода        |
| **[[static]] метод**                  | Direct              | В классах и структурах  |
| **[[class]] final метод**             | Direct              | В классах               |
| **[[private]] метод**                 | Direct              | Внутри файла            |
| **[[fileprivate]] метод**             | Direct              | Внутри файла            |
| **[[internal]] (если тип не открыт)** | Direct (часто)      | Оптимизация компилятора |
| **Метод в [[extension]]**             | Direct              | Даже для классов        |
| **[[Generic]] функция**               | Direct              | Для конкретного типа    |
| **Обычный метод класса**              | Table               | Для полиморфизма        |
| **[[@objc]] / dynamic**               | Message             | Для Obj-C совместимости |
| **[[Protocol]] requirement**          | Witness             | Через witness table     |

---

### Direct Dispatch vs Table Dispatch: визуализация

```mermaid
flowchart TD
    subgraph "Direct Dispatch"
        A[Вызов метода] --> B[Компилятор знает адрес]
        B --> C[Прямой jump к коду]
        C --> D[Выполнение]
    end
    
    subgraph "Table Dispatch (vtable)"
        E[Вызов метода] --> F[Чтение vtable из объекта]
        F --> G[Поиск адреса в таблице]
        G --> H[Косвенный jump]
        H --> I[Выполнение]
    end
```

---

### Производительность: пример измерения

```swift
import Darwin

class Test {
    // Table Dispatch
    func tableMethod() { }
    
    // Direct Dispatch
    final func directMethod() { }
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

let test = Test()
measure("Table", iterations: 10_000_000) {
    test.tableMethod()
}

measure("Direct", iterations: 10_000_000) {
    test.directMethod()
}

// Примерный результат:
// Table: 3.5 нс
// Direct: 1.2 нс
```

---

### Оптимизации для Direct Dispatch

#### 1. **Используйте final для классов, которые не наследуются**

```swift
// ✅ Хорошо
final class FastCalculator {
    func calculate() { }
}

// ❌ Плохо — если не планируется наследование
class Calculator {
    func calculate() { }  // Table Dispatch
}
```

#### 2. **Используйте структуры вместо классов**

```swift
// ✅ Хорошо — все методы Direct
struct Point {
    func distance() { }
}

// ❌ Плохо — если не нужна ссылочная семантика
class PointClass {
    func distance() { }  // Table Dispatch
}
```

#### 3. **Помечайте методы final в классах**

```swift
class Service {
    // Direct Dispatch
    final func criticalPath() { }
    
    // Table Dispatch (полиморфизм)
    func optionalOverride() { }
}
```

#### 4. **Выносите методы в extension**

```swift
class User {
    var name: String
    init(name: String) { self.name = name }
}

// Direct Dispatch
extension User {
    func greet() {
        print("Hello, \(name)")
    }
}
```

#### 5. **Используйте generics вместо existential**

```swift
// ❌ Existential — Witness Table
func process(_ shape: any Drawable) {
    shape.draw()
}

// ✅ Generic — Direct Dispatch
func process<T: Drawable>(_ shape: T) {
    shape.draw()
}
```

---

### Когда Direct Dispatch невозможен

Direct Dispatch невозможен, когда:

1.  **Требуется полиморфизм** — метод должен вызываться для разных типов.
2.  **Метод переопределен в подклассе** — нужна динамическая диспетчеризация.
3.  **Используется existential ([[any Protocol]])** — тип неизвестен до выполнения.
4.  **Метод помечен `@objc`** — требуется совместимость с [[Objective-C]] [[Runtime]].
5.  **Метод является требованием протокола** — вызов через witness table.

```swift
protocol Drawable {
    func draw()  // всегда через witness table (dynamic)
}

func drawShape(_ shape: any Drawable) {
    shape.draw()  // не может быть Direct — тип unknown
}
```

---

### Swift 6 и Direct Dispatch

Swift 6 усиливает возможности статической диспетчеризации:

- **Строгая типизация** — больше возможностей для статического анализа .
- **@preconcurrency** — помогает компилятору определить статические вызовы .
- **Оптимизации в release сборке** — агрессивный инлайнинг .
- **Неявная статическая диспетчеризация** для некоторых классов, если компилятор может доказать отсутствие переопределений .

```swift
// Swift 6 — компилятор может оптимизировать даже обычные методы
// если докажет, что переопределения нет
class Service {
    func process() { }  // может быть оптимизирован в Direct
}
```

---

### Короткое правило

> **Direct Dispatch** — самый быстрый вид диспетчеризации в [[Swift]].  
> Используйте **`final`**, **`static`**, **структуры**, **`private`**, **`some`** и **`generics`**  
> чтобы получить максимальную производительность в горячих путях.

### Итог

**Direct Dispatch** — фундаментальный механизм оптимизации в Swift:

1.  **Самый быстрый вызов** (~1–2 нс) — без динамического поиска .
2.  **Позволяет инлайнинг** — компилятор может встроить код метода .
3.  **Применяется для**:
    - `struct` и `enum`
    - `final` классы и методы
    - `static` и `class final` методы
    - `private` / `fileprivate` методы
    - Методы в `extension`
    - Generic функции
    - Opaque types (`some`)
4.  **Ограничения** — нет полиморфизма, нельзя переопределить .
5.  **Оптимизация** — критична для горячих путей (циклы, рендеринг, аудио) .

Выбор Direct Dispatch там, где это возможно, значительно улучшает производительность и предсказуемость кода .