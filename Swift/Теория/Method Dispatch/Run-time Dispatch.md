#metod_dispatch #Swift
**Run-time Dispatch** (также **Dynamic Dispatch**, **Late Binding**, **виртуальная диспетчеризация**) — это механизм, при котором **конкретная реализация метода** определяется **во время выполнения программы** ([[runtime]]), а не на этапе компиляции.

Это позволяет реализовать **полиморфизм**: одна и та же переменная/ссылка может указывать на разные типы, и при вызове метода будет выполнена **реализация того типа**, который реально лежит в памяти.

### 2. Когда в [[Swift]] включается Run-time Dispatch

| Условие / Конструкция                             | Тип диспетчеризации         | Почему runtime               | Скорость | Пример                        |
| ------------------------------------------------- | --------------------------- | ---------------------------- | -------- | ----------------------------- |
| Обычный метод класса (не [[final]])               | vtable ([[Table Dispatch]]) | Может быть переопределён     | ★★★★☆    | `func speak()` в классе       |
| Метод протокола, вызванный через [[any Protocol]] | Witness Table               | Тип неизвестен компилятору   | ★★★★☆    | `let s: any Shape = Circle()` |
| `@objc` / `dynamic` метод                         | [[Message Dispatch]]        | Obj-C runtime (objc_msgSend) | ★★☆☆☆    | `@objc func didTap()`         |
| `override func` в подклассе                       | vtable                      | Динамический выбор           | ★★★★☆    | `override func viewDidLoad()` |

### 3. Основные механизмы Run-time Dispatch в Swift

| Механизм                              | Когда используется                     | Таблица / Поиск | Скорость (нс, примерно) | Overhead |
| ------------------------------------- | -------------------------------------- | --------------- | ----------------------- | -------- |
| **vtable** (Class [[Table Dispatch]]) | Обычные не-final методы классов        | vtable          | ~3–5 нс                 | Средний  |
| **Witness Table**                     | Протоколы без `@objc` (any / generics) | witness table   | ~3–7 нс                 | Средний  |
| **[[Message Dispatch]]**              | `@objc`, `dynamic`, NSObject-подклассы | objc_msgSend    | ~10–20 нс               | Большой  |

### 4. Полные примеры кода

#### Пример 1 — Run-time Dispatch через классы (vtable)

```swift
class Animal {
    func speak() {                      // vtable entry
        print("Generic animal sound")
    }
}

class Dog: Animal {
    override func speak() {
        print("Woof!")
    }
}

class Cat: Animal {
    override func speak() {
        print("Meow!")
    }
}

let pets: [Animal] = [Dog(), Cat(), Animal()]
pets.forEach { $0.speak() }
// Woof!
// Meow!
// Generic animal sound
```

**Здесь** каждый вызов идёт через vtable реального типа объекта.

#### Пример 2 — Run-time Dispatch через протокол (witness table)

```swift
protocol Speaker {
    func speak()
}

struct Parrot: Speaker {
    func speak() { print("Hello!") }
}

class Robot: Speaker {
    func speak() { print("Beep boop") }
}

let speakers: [any Speaker] = [Parrot(), Robot()]
speakers.forEach { $0.speak() }
// Hello!
// Beep boop
```

**Witness table** для Parrot и Robot используется в runtime.

#### Пример 3 — Сравнение с [[compile-time dispatch]]

```swift
struct Bird {
    func chirp() { print("Chirp!") }     // compile-time
}

let bird = Bird()
bird.chirp()  // прямой вызов — компилятор знает адрес

// Контраст:
let speaker: any Speaker = Parrot()
speaker.speak()  // run-time через witness table
```

#### Пример 4 — @objc протокол ([[Message Dispatch]])

```swift
@objc protocol Notifier {
    func notify()
}

class Phone: NSObject, Notifier {
    func notify() { print("Ring ring!") }
}

let notifier: Notifier = Phone()
notifier.notify()  // objc_msgSend → Message Dispatch
```

### 5. Сравнение производительности (примерные цифры 2026)

| Тип диспетчеризации       | Вызов метода (нс) | Разница с Static | Где критично |
|----------------------------|-------------------|-------------------|--------------|
| Static / Direct            | ~1–2 нс           | 1×                | Горячие циклы, UI, ML |
| Table Dispatch (vtable)    | ~3–5 нс           | 2–3× медленнее    | Обычные классы |
| Witness Table (any Protocol) | ~4–7 нс         | 3–4× медленнее    | Коллекции протоколов |
| Message Dispatch (@objc)   | ~10–20 нс         | 5–10× медленнее   | Редко |

### 6. Лучшие практики и рекомендации 2026 (Swift 6+)

- **Избегайте `any Protocol` в горячих путях** — используйте `some Protocol` или generics `<T: Protocol>`
- **final** — отключает динамику и переводит в Static Dispatch
- **[[some Protocol]]** — возвращаемый тип функций → максимальная скорость
- **@objc / dynamic** — только для KVO, delegates, старый UIKit, совместимость с Obj-C
- **[[SwiftUI]]** — `some View`, `some ViewModel` — стандарт (часто Static / Witness)
- **Горячие пути** (рендеринг, 60 fps, ML-инференс) — минимизируйте `any` и `@objc`
- **Swift 6 strict concurrency** — `any` усложняет проверку потокобезопасности → используйте `some` и generics

**Короткий девиз 2026**:
> «Run-time Dispatch — это магия полиморфизма: когда компилятор говорит «я не знаю, что там лежит, разберёмся на месте».  
> [[some]], [[final]], [[generic]] — для скорости.  
> [[any]] и [[@objc]] — только когда без динамики никак.»
