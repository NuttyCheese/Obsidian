**Direct Dispatch** (также называется **Static Dispatch** или **прямая диспетчеризация**) — это механизм вызова методов/функций, при котором компилятор **на этапе компиляции** знает точный адрес вызываемой функции и генерирует **прямой вызов** (без таблиц, без поиска, без [[Runtime]]-проверок).

Это **самый быстрый** способ диспетчеризации в Swift.

### 2. Когда в Swift включается Direct Dispatch

Direct Dispatch происходит **автоматически** в следующих случаях:

| Условие / Конструкция                                          | Тип диспетчеризации | Почему компилятор знает адрес заранее |
| -------------------------------------------------------------- | ------------------- | ------------------------------------- |
| `final func` в классе                                          | Direct              | Метод нельзя переопределить           |
| `static func` / `class func`                                   | Direct              | Статический метод, нет полиморфизма   |
| [[private]] / [[fileprivate]] метод                            | Direct              | Никто снаружи не может переопределить |
| Метод в [[struct]] / [[enum]] (без протокола)                  | Direct              | Value type → нет vtable               |
| Метод в протоколе, но вызывается на конкретном типе ([[some]]) | Direct / Witness    | Компилятор знает точный тип           |
| [[let]] / [[var]] с конкретным типом (не [[any]])              | Direct              | Тип известен полностью                |

### 3. Схема: Direct Dispatch vs другие виды

```mermaid
graph TD
    A[Вызов метода] --> B{Тип диспетчеризации?}

    B -->|Direct Dispatch| C[Прямой вызов по адресу<br>Компилятор знает всё заранее<br>★ Максимальная скорость ★]
    B -->|Table Dispatch| D[vtable / witness table<br>1–2 лишние инструкции<br>★★★ Высокая скорость ★★★]
    B -->|Message Dispatch| E[objc_msgSend<br>Поиск в кэше/таблице<br>★ Самая медленная ★]

    C --> F[final func, static func, private, struct/enum]
    D --> G[override func в классе, протоколы без @objc]
    E --> H[@objc, dynamic, NSObject]
```

### 4. Примеры кода

#### Пример 1 — final метод (чистый Direct Dispatch)

```swift
class Animal {
    final func makeSound() {            // final → Direct Dispatch
        print("Generic sound")
    }
}

class Dog: Animal {
    // override func makeSound() {}     // Ошибка компиляции
}

let dog: Animal = Dog()
dog.makeSound()  // прямой вызов, компилятор знает адрес
```

#### Пример 2 — static метод

```swift
class Math {
    static func square(_ x: Int) -> Int {  // static → Direct Dispatch
        return x * x
    }
}

print(Math.square(5))  // прямой вызов
```

#### Пример 3 — private метод

```swift
class Counter {
    private var count = 0
    
    private func increment() {   // private → Direct Dispatch
        count += 1
    }
    
    func publicIncrement() {
        increment()  // прямой вызов внутри класса
    }
}
```

#### Пример 4 — struct / enum (всегда Direct, если нет протокола)

```swift
struct Point {
    var x: Int
    var y: Int
    
    func distance() -> Double {   // Direct Dispatch
        return sqrt(Double(x*x + y*y))
    }
}

var p = Point(x: 3, y: 4)
print(p.distance())  // 5.0 — прямой вызов
```

#### Пример 5 — some Protocol (часто Direct или Witness)

```swift
protocol Renderer {
    func render()
}

struct Circle: Renderer {
    func render() { print("○") }
}

func makeRenderer() -> some Renderer {
    Circle()  // some → компилятор знает тип → часто Direct / Witness
}

let r = makeRenderer()
r.render()  // высокая скорость
```

### 5. Сравнение производительности (примерные цифры 2026)

| Тип диспетчеризации       | Вызов метода (нс) | Разница с Direct | Где критично |
|----------------------------|-------------------|-------------------|--------------|
| Direct Dispatch            | ~1–2 нс           | 1×                | Горячие циклы, UI-рендеринг |
| Table Dispatch (vtable/witness) | ~3–5 нс     | 2–3× медленнее    | Большинство кода |
| Message Dispatch (objc_msgSend) | ~10–20 нс | 5–10× медленнее   | Редко, только @objc |

### 6. Реальные сценарии в iOS-разработке 2026

#### Сценарий 1 — [[SwiftUI]] (часто Direct / Witness)

```swift
struct ContentView: View {
    var body: some View {           // some → Direct / Witness
        Text("Hello, world!")
    }
}
```

#### Сценарий 2 — Оптимизация производительности

```swift
final class FastRenderer {
    final func renderFrame() {      // Direct Dispatch
        // горячий путь — рендеринг 60 fps
    }
}
```

#### Сценарий 3 — Протоколы и generics (максимальная скорость)

```swift
func process<T: Equatable>(_ items: [T]) {
    // T известен → Direct Dispatch внутри
}
```

### 7. Лучшие практики 2026 (Swift 6 strict concurrency)

- Делайте методы **final** везде, где полиморфизм не нужен  
- Используйте **static** для утилитарных методов  
- Для протоколов — **[[some]]** при возврате из функций  
- [[any Protocol]] — только для коллекций и свойств  
- `@objc dynamic` — только для [[KVO]], [[Delegate]], старый [[UIKit]]  
- В SwiftUI — `some View`, `some ViewModel` — стандарт  
- В горячих путях (UI, рендеринг, циклы) — избегайте `any` и `@objc`  
- Для максимальной скорости — generics `<T: Protocol>` вместо `any`

**Короткий девиз 2026**:
> «Direct Dispatch — это когда компилятор говорит: «Я знаю, что вызывать, и сделаю это максимально быстро».  
> final, static, private, some — ваши лучшие друзья для скорости.  
> any и @objc — только когда без них действительно нельзя.»
