#metod_dispatch #Swift
**Compile-time Dispatch** (статическая диспетчеризация, также известна как **[[Direct Dispatch]]** или **прямая диспетчеризация**) — это механизм в [[Swift]], при котором компилятор **на этапе компиляции** точно знает, **какая именно реализация** метода или функции будет вызвана, и генерирует **прямой вызов** по фиксированному адресу в памяти.

Это **самый быстрый** способ диспетчеризации в Swift, потому что:

- нет поиска в таблицах (vtable / witness table)  
- нет objc_msgSend  
- нет runtime-проверок  
- часто возможен **инлайн** (встраивание кода функции прямо в место вызова)

### Когда в Swift используется Compile-time Dispatch (2026)

| Условие / Конструкция                              | Тип диспетчеризации | Почему компилятор знает адрес заранее      | Скорость | Пример                                 |
| -------------------------------------------------- | ------------------- | ------------------------------------------ | -------- | -------------------------------------- |
| Метод в [[struct]] / [[enum]] (без протокола)      | Static / Direct     | [[Value type]] → нет наследования и vtable | ★★★★★    | `struct Point { func distance() }`     |
| `final func` в классе                              | Static / Direct     | Нельзя переопределить                      | ★★★★★    | `final func compute()`                 |
| `static func` / `class func`                       | Static / Direct     | Статический контекст                       | ★★★★★    | `static func square(_ x: Int)`         |
| [[private]] / [[fileprivate]] метод                | Static / Direct     | Никто снаружи не может переопределить      | ★★★★★    | `private func internalHelper()`        |
| Метод протокола, вызванный через [[some Protocol]] | Static / Witness    | Компилятор знает конкретный тип            | ★★★★☆    | `func render<T: Drawable>(_ shape: T)` |
| [[let]] / [[var]] с конкретным типом (не [[any]])  | Static / Direct     | Тип полностью известен                     | ★★★★★    | `let obj = MyStruct()`                 |

### Полные примеры кода

#### 1. [[Struct]] / [[Enum]] — всегда [[Static Dispatch]] (если нет протокола)

```swift
struct Counter {
    private var count = 0
    
    mutating func increment() {     // Static Dispatch
        count += 1
    }
    
    func value() -> Int {           // Static Dispatch
        return count
    }
}

var c = Counter()
c.increment()
print(c.value())  // прямой вызов, компилятор знает адрес
```

#### 2. [[final]] метод класса — отключает vtable

```swift
class FastCalculator {
    final func square(_ x: Int) -> Int {   // final → Static Dispatch
        return x * x
    }
}

let calc = FastCalculator()
print(calc.square(7))  // прямой вызов — нет vtable
```

#### 3. static / [[class]] методы — всегда Static

```swift
class Logger {
    static func info(_ message: String) {   // static → Static Dispatch
        print("[INFO] \(message)")
    }
}

Logger.info("App started")  // прямой вызов
```

#### 4. [[some Protocol]] — часто [[Static Dispatch]]

```swift
protocol Drawable {
    func draw()
}

struct Circle: Drawable {
    func draw() { print("○") }
}

func renderShape(_ shape: some Drawable) {   // some → компилятор знает тип Circle
    shape.draw()  // Static / Witness → очень быстро
}

let circle = Circle()
renderShape(circle)  // ○
```

#### 5. Контраст с [[Dynamic Dispatch]] (для понимания разницы)

```swift
class Animal {
    func makeSound() { print("Generic") }  // vtable → Dynamic
}

class Dog: Animal {
    override func makeSound() { print("Woof") }
}

let animal: Animal = Dog()
animal.makeSound()  // Dynamic: Woof (через vtable)

final class FastAnimal {
    final func makeSound() { print("Fast generic") }  // Direct
}

let fast: FastAnimal = FastAnimal()
fast.makeSound()  // Static: прямой вызов
```

### Сравнение производительности (примерные цифры 2026)

| Тип диспетчеризации       | Вызов метода (нс) | Разница с Static | Где критично |
|----------------------------|-------------------|-------------------|--------------|
| Static / Direct            | ~1–2 нс           | 1×                | 60 fps UI, ML-инференс, аудио, горячие циклы |
| Table Dispatch (vtable/witness) | ~3–5 нс     | 2–3× медленнее    | Большинство кода |
| Message Dispatch (objc_msgSend) | ~10–20 нс | 5–10× медленнее   | Редко, только @objc |

### Реальные сценарии в iOS-разработке 2026

#### Сценарий 1 — SwiftUI (часто Static / Witness)

```swift
struct ContentView: View {
    var body: some View {           // some → Static / Witness
        Text("Hello, world!")
            .font(.largeTitle)
    }
}
```

#### Сценарий 2 — Оптимизация рендеринга / обработки кадров

```swift
final class MetalRenderer {
    final func renderFrame(buffer: MTLBuffer) {  // Direct → максимальная скорость
        // 60–120 fps
    }
}
```

#### Сценарий 3 — Утилитарные функции ([[String]], [[Date]], Math)

```swift
extension String {
    static func isValidEmail(_ email: String) -> Bool {  // static → Static Dispatch
        // regex / NSPredicate
        return true
    }
}
```

### Лучшие практики и оптимизации 2026 (Swift 6 strict concurrency)

- Делайте методы **final** везде, где полиморфизм не нужен  
- Используйте **static** для утилитарных методов  
- В протоколах — возвращайте **some Protocol**  
- `private` / `fileprivate` — автоматический Static Dispatch  
- В [[SwiftUI]] — `some View`, `some ViewModel` — стандарт  
- В горячих путях — избегайте `any` и `@objc dynamic`  
- Для максимальной скорости — generics `<T: Protocol>` вместо `any`  
- В Swift 6 — Static Dispatch сильно упрощает проверку потокобезопасности

**Короткий девиз 2026**:
> «Static Dispatch — это когда компилятор говорит: «Я уже всё посчитал и сделаю это мгновенно».  
> final, static, private, some — ваши лучшие друзья для скорости и безопасности.  
> any и @objc — только когда без них действительно нельзя.»
