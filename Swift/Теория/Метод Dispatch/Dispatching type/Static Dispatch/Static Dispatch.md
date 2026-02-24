**Static Dispatch** (также называется **[[[Direct Dispatch]]]** или **прямая диспетчеризация**) — это механизм, при котором компилятор **на этапе компиляции** точно знает, какой именно метод или функция будет вызвана, и генерирует **прямой вызов** по фиксированному адресу.

Это **самый быстрый** способ вызова в Swift, потому что:

- нет поиска в таблице (vtable/witness table)  
- нет objc_msgSend  
- нет runtime-проверок  

Компилятор просто вставляет инструкцию `call` на нужный адрес — как обычный вызов функции в C.

### 2. Когда в Swift включается Static Dispatch

| Конструкция / Условие                                                | Тип диспетчеризации | Почему компилятор знает адрес заранее |
| -------------------------------------------------------------------- | ------------------- | ------------------------------------- |
| `final func` в классе                                                | Static / Direct     | Метод нельзя переопределить           |
| `static func` / `class func`                                         | Static / Direct     | Статический метод, нет полиморфизма   |
| [[private]] / [[fileprivate]] / [[internal]] (если нет [[override]]) | Static / Direct     | Никто снаружи не может переопределить |
| Метод в [[struct]] / [[enum]] (без протокола)                        | Static / Direct     | [[Value Type]] → нет vtable           |
| Метод в протоколе, но вызывается на [[some Protocol]]                | Static / Witness    | Компилятор знает точный тип           |
| [[let]] / [[var]] с конкретным типом (не [[any]])                    | Static / Direct     | Тип известен полностью                |

### 3. Примеры кода — от простого к продвинутому

#### Пример 1 — final метод в классе (чистый Static Dispatch)

```swift
class Animal {
    final func makeSound() {            // final → Static Dispatch
        print("Generic sound")
    }
}

class Dog: Animal {
    // override func makeSound() {}     // Ошибка компиляции
}

let animal: Animal = Dog()
animal.makeSound()  // прямой вызов — компилятор знает адрес функции Animal.makeSound
```

#### Пример 2 — static метод (всегда Static Dispatch)

```swift
class MathUtils {
    static func square(_ x: Int) -> Int {   // static → Static Dispatch
        return x * x
    }
}

print(MathUtils.square(7))  // прямой вызов
```

#### Пример 3 — [[private]] / [[fileprivate]] метод

```swift
class Counter {
    private var count = 0
    
    private func increment() {   // private → Static Dispatch
        count += 1
    }
    
    func publicIncrement() {
        increment()  // внутри класса — прямой вызов
    }
}
```

#### Пример 4 — struct / enum (почти всегда Static Dispatch)

```swift
struct Point {
    var x: Double
    var y: Double
    
    func distanceToOrigin() -> Double {   // Static Dispatch
        return sqrt(x*x + y*y)
    }
}

var p = Point(x: 3, y: 4)
print(p.distanceToOrigin())  // прямой вызов
```

#### Пример 5 — some Protocol (часто Static Dispatch)

```swift
protocol Renderer {
    func render()
}

struct Circle: Renderer {
    func render() { print("○") }
}

func makeRenderer() -> some Renderer {   // some → компилятор знает тип Circle
    Circle()
}

let r = makeRenderer()
r.render()  // часто прямой вызов (или witness table, но очень быстро)
```

### 4. Сравнение всех видов диспетчеризации в Swift 2026

| Тип диспетчеризации       | Скорость | Размер кода | Полиморфизм | Переопределение | Примеры |
|---------------------------|----------|-------------|-------------|------------------|---------|
| **Static / Direct**       | ★★★★★    | Минимальный | Нет         | Нет              | `final`, `static`, `private`, struct/enum |
| Table (vtable)            | ★★★★☆    | Средний     | Да          | Да               | Обычные методы классов |
| Witness Table             | ★★★★☆    | Средний     | Да          | Да               | Протоколы без @objc |
| Message (objc_msgSend)    | ★★☆☆☆    | Большой     | Да          | Да               | `@objc`, `dynamic`, NSObject |

### 5. Производительность (примерные цифры 2026)

| Тип диспетчеризации       | Вызов метода (нс) | Разница с Static | Где критично |
|----------------------------|-------------------|-------------------|--------------|
| Static / Direct            | ~1–2 нс           | 1×                | Горячие циклы, UI-рендеринг, обработка кадров |
| Table Dispatch             | ~3–5 нс           | 2–3× медленнее    | Большинство кода |
| Message Dispatch           | ~10–20 нс         | 5–10× медленнее   | Редко, только @objc |

**Вывод**:  
Static Dispatch в 3–10 раз быстрее динамических видов.  
В горячих путях (60 fps UI, обработка аудио/видео, ML-инференс) — старайтесь использовать именно его.

### 6. Реальные сценарии в iOS-разработке 2026

#### Сценарий 1 — Оптимизация рендеринга (SwiftUI / Metal)

```swift
final class MetalRenderer {
    final func renderFrame(buffer: MTLBuffer) {
        // горячий путь — 60–120 fps
        // прямой вызов → минимальная задержка
    }
}
```

#### Сценарий 2 — Утилитарные методы (Math, [[String]], [[Date]])

```swift
extension String {
    static func isValidEmail(_ email: String) -> Bool {  // static → Static Dispatch
        // regex или NSPredicate
        return true
    }
}
```

#### Сценарий 3 — SwiftUI body (some View)

```swift
struct ContentView: View {
    var body: some View {           // some → часто Static Dispatch
        Text("Hello, world!")
            .font(.largeTitle)
    }
}
```

### 7. Лучшие практики и оптимизации 2026 (Swift 6+)

- Делайте методы **final** везде, где полиморфизм не нужен  
- Используйте **static** для утилитарных методов (Math, [[DateFormatter]] и т.д.)  
- В протоколах — **some Protocol** при возврате из функций  
- `private` / `fileprivate` — автоматический Static Dispatch  
- В [[SwiftUI]] — `some View`, `some ViewModel` — стандарт  
- В горячих путях — избегайте `any` и `@objc dynamic`  
- Для максимальной скорости — generics `<T: Protocol>` вместо `any`  
- В [[Swift]] 6 strict concurrency — Static Dispatch упрощает проверку потокобезопасности

**Короткий девиз 2026**:
> «Static Dispatch — это когда компилятор говорит: «Я уже всё знаю и сделаю это мгновенно».  
> final, static, private, some — ваши лучшие друзья для скорости и безопасности.  
> any и @objc — только когда без них действительно нельзя.»
