#swift #dispatch #static-dispatch #performance #optimization #swift6 #compile-time

---
### Определение
**Static Dispatch (статическая диспетчеризация)** — это механизм вызова методов, при котором **компилятор** на этапе компиляции точно определяет, какая реализация метода будет вызвана, и заменяет вызов прямым переходом к коду метода . Это самый быстрый и предсказуемый способ вызова функций в [[Swift]].

В отличие от динамической диспетчеризации ([[Table Dispatch]], [[Message Dispatch]]), статическая диспетчеризация не требует поиска в таблицах во время выполнения, что делает ее значительно быстрее и позволяет компилятору выполнять агрессивные оптимизации, такие как инлайнинг .

### Зачем это знать iOS-разработчику?
1.  **Максимальная производительность:** Static Dispatch — самый быстрый вид диспетчеризации (~1–2 нс на вызов) .
2.  **Инлайнинг:** Компилятор может встроить код метода в место вызова, устраняя overhead полностью .
3.  **Оптимизация горячих путей:** Критичен для циклов, рендеринга, аудиообработки, ML-инференса .
4.  **Предсказуемость:** Нет динамического поиска, что упрощает отладку и анализ .
5.  **Swift 6 strict concurrency:** Статическая диспетчеризация упрощает проверку потокобезопасности .

---

### Static Dispatch vs Dynamic Dispatch

| Характеристика        | Static Dispatch                                          | Dynamic Dispatch                                     |
| --------------------- | -------------------------------------------------------- | ---------------------------------------------------- |
| **Время определения** | Компиляция                                               | Выполнение                                           |
| **Скорость**          | ★★★★★ (~1–2 нс)                                          | ★★★☆☆ (~3–20 нс)                                     |
| **Полиморфизм**       | Нет                                                      | Да                                                   |
| **Переопределение**   | Нет                                                      | Да                                                   |
| **Инлайнинг**         | Да (часто)                                               | Редко                                                |
| **Overhead**          | Минимальный                                              | Средний / Высокий                                    |
| **Где используется**  | [[struct]], [[enum]], [[final]], [[static]], [[private]] | [[class]] (обычные методы), [[protocol]]s, [[@objc]] |

---

### Когда используется Static Dispatch

#### 1. **Структуры и перечисления**

```swift
struct Point {
    var x: Int
    var y: Int
    
    // Все методы struct — Static Dispatch
    func distance() -> Double {
        return sqrt(Double(x * x + y * y))
    }
}

enum Status {
    case success, failure
    
    // Все методы enum — Static Dispatch
    func isSuccess() -> Bool {
        return self == .success
    }
}

let point = Point(x: 3, y: 4)
point.distance()  // Static Dispatch
```

#### 2. **Методы, помеченные [[final]]**

```swift
class Animal {
    // final — нельзя переопределить → Static Dispatch
    final func breathe() {
        print("Breathing")
    }
    
    // Обычный метод → Dynamic (Table) Dispatch
    func move() {
        print("Moving")
    }
}

let animal = Animal()
animal.breathe()  // Static Dispatch
animal.move()     // Dynamic Dispatch
```

#### 3. **Методы [[static]] и [[class]] final**

```swift
class Math {
    // static → Static Dispatch
    static func square(_ x: Int) -> Int {
        return x * x
    }
    
    // class final → Static Dispatch
    class final func cube(_ x: Int) -> Int {
        return x * x * x
    }
}

Math.square(5)   // Static Dispatch
Math.cube(3)     // Static Dispatch
```

#### 4. **Методы в [[extension]]s**

```swift
class Person {
    var name: String
    init(name: String) { self.name = name }
}

// Extension методы — Static Dispatch
extension Person {
    func greet() {
        print("Hello, I'm \(name)")
    }
}

let person = Person(name: "Alice")
person.greet()  // Static Dispatch

class Student: Person {
    // ❌ Нельзя override extension метод
    // override func greet() { }  // Ошибка!
}
```

#### 5. **Приватные и [[fileprivate]] методы**

```swift
class Service {
    // private → Static Dispatch (внутри файла)
    private func helper() {
        print("helper")
    }
    
    // fileprivate → Static Dispatch (внутри файла)
    fileprivate func internalHelper() {
        print("internal")
    }
    
    func process() {
        helper()  // Static Dispatch
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

// Generic функция → Static Dispatch
func drawShape<T: Drawable>(_ shape: T) {
    shape.draw()  // Static Dispatch (тип известен на этапе компиляции)
}

let circle = Circle()
drawShape(circle)  // Static вызов Circle.draw()
```

#### 7. **Opaque types ([[some]])**

```swift
func makeDrawable() -> some Drawable {
    return Circle()  // конкретный тип скрыт, но известен компилятору
}

let shape = makeDrawable()
shape.draw()  // Static Dispatch (Circle.draw())
```

---

### Как компилятор обрабатывает Static Dispatch

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

### Static Dispatch и инлайнинг

Компилятор может **инлайнить** методы, вызываемые через Static Dispatch, полностью устраняя [[overhead]] вызова.

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

**Пример инлайнинга в SIL:**
```swift
// Без инлайнинга:
%1 = function_ref @Math.square
%2 = apply %1(5)     // вызов функции

// С инлайнингом:
%1 = 5 * 5            // код встроен
```

---

### Правила определения Static Dispatch

| Конструкция                  | Вид диспетчеризации | Примечание              |
| ---------------------------- | ------------------- | ----------------------- |
| **[[struct]] / [[enum]]**    | Static              | Все методы              |
| **[[final]] [[class]]**      | Static              | Все методы              |
| **final метод в классе**     | Static              | Для этого метода        |
| **[[static]] метод**         | Static              | В классах и структурах  |
| **class final метод**        | Static              | В классах               |
| **[[private]] метод**        | Static              | Внутри файла            |
| **[[fileprivate]] метод**    | Static              | Внутри файла            |
| **Метод в [[extension]]**    | Static              | Даже для классов        |
| **[[Generic]] функция**      | Static              | Для конкретного типа    |
| **Обычный метод класса**     | Dynamic (Table)     | Для полиморфизма        |
| **@objc / dynamic**          | Dynamic (Message)   | Для Obj-C совместимости |
| **[[Protocol]] requirement** | Dynamic (Witness)   | Через witness table     |

---

### Производительность: пример измерения

```swift
import Darwin

class Test {
    // Dynamic Dispatch
    func dynamicMethod() { }
    
    // Static Dispatch
    final func staticMethod() { }
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
measure("Dynamic", iterations: 10_000_000) {
    test.dynamicMethod()
}

measure("Static", iterations: 10_000_000) {
    test.staticMethod()
}

// Примерный результат:
// Dynamic: 3.5 нс
// Static: 1.2 нс
```

---

### Оптимизации для Static Dispatch

#### 1. **Используйте final для классов, которые не наследуются**

```swift
// ✅ Хорошо
final class FastCalculator {
    func calculate() { }
}

// ❌ Плохо — если не планируется наследование
class Calculator {
    func calculate() { }  // Dynamic Dispatch
}
```

#### 2. **Используйте структуры вместо классов**

```swift
// ✅ Хорошо — все методы Static
struct Point {
    func distance() { }
}

// ❌ Плохо — если не нужна ссылочная семантика
class PointClass {
    func distance() { }  // Dynamic Dispatch
}
```

#### 3. **Помечайте методы final в классах**

```swift
class Service {
    // Static Dispatch
    final func criticalPath() { }
    
    // Dynamic Dispatch (полиморфизм)
    func optionalOverride() { }
}
```

#### 4. **Выносите методы в extension**

```swift
class User {
    var name: String
    init(name: String) { self.name = name }
}

// Static Dispatch
extension User {
    func greet() {
        print("Hello, \(name)")
    }
}
```

#### 5. **Используйте generics вместо existential**

```swift
// ❌ Existential — Dynamic (Witness Table)
func process(_ shape: any Drawable) {
    shape.draw()
}

// ✅ Generic — Static Dispatch
func process<T: Drawable>(_ shape: T) {
    shape.draw()
}
```

#### 6. **Используйте some для скрытия типа**

```swift
// ❌ Existential — Dynamic
func makeDrawable() -> any Drawable {
    return Circle()
}

// ✅ Opaque type — Static
func makeDrawable() -> some Drawable {
    return Circle()
}
```

---

### Когда Static Dispatch невозможен

Static Dispatch невозможен, когда:

1.  **Требуется полиморфизм** — метод должен вызываться для разных типов.
2.  **Метод переопределен в подклассе** — нужна динамическая диспетчеризация.
3.  **Используется existential (`any Protocol`)** — тип неизвестен до выполнения.
4.  **Метод помечен `@objc`** — требуется совместимость с [[Objective-C]] [[runtime]].
5.  **Метод является требованием протокола** — вызов через witness table.

```swift
protocol Drawable {
    func draw()  // всегда через witness table (dynamic)
}

func drawShape(_ shape: any Drawable) {
    shape.draw()  // не может быть Static — тип unknown
}
```

---

### Swift 6 и Static Dispatch

Swift 6 усиливает возможности статической диспетчеризации:

- **Строгая типизация** — больше возможностей для статического анализа .
- **@preconcurrency** — помогает компилятору определить статические вызовы .
- **Оптимизации в release сборке** — агрессивный инлайнинг .
- **Неявная статическая диспетчеризация** для некоторых классов, если компилятор может доказать отсутствие переопределений .

```swift
// Swift 6 — компилятор может оптимизировать даже обычные методы
// если докажет, что переопределения нет
class Service {
    func process() { }  // может быть оптимизирован в Static
}
```

---

### Короткое правило

> **Static Dispatch** — самый быстрый вид диспетчеризации в Swift.  
> Используйте **`final`**, **`static`**, **структуры**, **`private`**, **`some`** и **`generics`**  
> чтобы получить максимальную производительность в горячих путях.

### Итог

**Static Dispatch** — фундаментальный механизм оптимизации в Swift:

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

Выбор Static Dispatch там, где это возможно, значительно улучшает производительность и предсказуемость кода .