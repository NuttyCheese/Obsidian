**Диспетчеризация** — это процесс определения, **какая именно реализация** метода (или свойства) будет вызвана при обращении к нему через переменную/ссылку.

Swift использует **четыре основных механизма** диспетчеризации:

| Механизм                                              | Время определения | Скорость | Полиморфизм | Переопределение | Размер кода | Примеры                                   |
| ----------------------------------------------------- | ----------------- | -------- | ----------- | --------------- | ----------- | ----------------------------------------- |
| **[[Direct Dispatch\|Direct]] / [[Static Dispatch]]** | Компиляция        | ★★★★★    | Нет         | Нет             | Минимальный | `final`, `static`, `private`, struct/enum |
| **[[Table Dispatch]] ([[vtable]])**                   | Выполнение        | ★★★★☆    | Да          | Да              | Средний     | Обычные методы классов                    |
| **[[Witness Table Dispatch]]**                        | Выполнение        | ★★★★☆    | Да          | Да              | Средний     | Протоколы без `@objc`                     |
| **[[Message Dispatch]]**                              | Выполнение        | ★★☆☆☆    | Да          | Да              | Большой     | `@objc`, `dynamic`, `NSObject`            |

### 2. Подробное сравнение (2026 актуально)

| Характеристика         | [[Direct Dispatch\|Direct]] / Static | [[Table Dispatch]] (vtable) | [[Witness Table Dispatch\|Witness Table]] | [[Message Dispatch]] (objc_msgSend) |
| ---------------------- | ------------------------------------ | --------------------------- | ----------------------------------------- | ----------------------------------- |
| Время определения      | Компиляция                           | [[Runtime]]                 | Runtime                                   | Runtime                             |
| Скорость вызова        | ~1–2 нс                              | ~3–5 нс                     | ~3–5 нс                                   | ~10–20 нс                           |
| Полиморфизм            | Нет                                  | Да                          | Да                                        | Да                                  |
| Поддержка override     | Нет                                  | Да                          | Да                                        | Да                                  |
| Поддержка struct/enum  | Да                                   | Нет                         | Да (протоколы)                            | Нет                                 |
| Поддержка протоколов   | Да ([[some]] / [[generic]]s)         | Нет                         | Да                                        | Да (если [[@objc]])                 |
| Overhead в памяти      | Минимальный                          | Средний                     | Средний                                   | Большой                             |
| Инлайнинг компилятором | Да (часто)                           | Редко                       | Редко                                     | Почти никогда                       |
| Где критично           | Горячие циклы, UI, ML                | Большинство кода            | Протоколы                                 | Редко (только Obj-C совместимость)  |

### 3. Подробные примеры кода

#### 3.1 Direct / Static Dispatch (самый быстрый)

```swift
struct Math {
    static func square(_ x: Int) -> Int {   // static → Direct
        return x * x
    }
}

final class FastCounter {
    final func increment() {                // final → Direct
        count += 1
    }
    private var count = 0
}

let c = FastCounter()
c.increment()  // прямой вызов — компилятор знает адрес
```

#### 3.2 Table Dispatch (vtable) — обычные методы классов

```swift
class Animal {
    func makeSound() {          // vtable → динамическая
        print("Generic")
    }
}

class Dog: Animal {
    override func makeSound() {
        print("Woof!")
    }
}

let pets: [Animal] = [Dog(), Animal()]
pets.forEach { $0.makeSound() }  // Woof! → Generic
```

#### 3.3 Witness Table Dispatch — протоколы без @objc

```swift
protocol Drawable {
    func draw()
}

struct Circle: Drawable {
    func draw() { print("○") }
}

let shapes: [any Drawable] = [Circle()]
shapes.forEach { $0.draw() }  // witness table → ○
```

#### 3.4 Message Dispatch — @objc / dynamic

```swift
class MyView: UIView {
    @objc dynamic func didTap() {   // objc_msgSend
        print("Tapped!")
    }
}
```

### 4. Как выбрать нужный тип диспетчеризации (рекомендации 2026)

| Сценарий                                   | Рекомендация                | Почему                                   |
| ------------------------------------------ | --------------------------- | ---------------------------------------- |
| Метод не должен переопределяться           | `final func`                | Максимальная скорость + инлайнинг        |
| Утилитарный метод / хелпер                 | `static func`               | Static Dispatch всегда                   |
| Метод только для внутреннего использования | `private func`              | Автоматический Static Dispatch           |
| [[SwiftUI]] / возвращаемый тип протокола   | `some Protocol`             | Компилятор знает тип → Static / Witness  |
| Коллекция разных реализаций                | `[any Protocol]`            | Только any позволяет хранить разные типы |
| Делегаты / [[KVO]] / старый [[UIKit]]      | `@objc` / `dynamic`         | Message Dispatch обязателен              |
| Горячие пути (рендеринг, ML, 60 fps)       | `final`, `static`, generics | Избегать any и @objc                     |

### 5. Производительность (примерные цифры 2026)

| Тип диспетчеризации       | Вызов метода (нс) | Разница с Direct | Где критично |
|----------------------------|-------------------|-------------------|--------------|
| Direct / Static            | ~1–2 нс           | 1×                | UI-рендеринг, аудио, ML-инференс |
| Table (vtable/witness)     | ~3–5 нс           | 2–3× медленнее    | Большинство кода |
| Message (objc_msgSend)     | ~10–20 нс         | 5–10× медленнее   | Редко (только @objc) |

### 6. Лучшие практики 2026 (Swift 6 strict concurrency)

- **[[final]]** и **static** — везде, где полиморфизм не нужен  
- **[[private]]** / **[[fileprivate]]** — автоматический Direct Dispatch  
- **some Protocol** — для возвращаемых типов (функции, computed properties)  
- **any Protocol** — только для коллекций, свойств, делегатов  
- **@objc dynamic** — только для [[KVO]], delegates, старый [[UIKit]]  
- В [[SwiftUI]] — `some View`, `some ViewModel` — стандарт  
- В горячих путях — избегайте `any` и `@objc`  
- [[generic]] `<T: Protocol>` — вместо `any` для статической диспетчеризации  
- В Swift 6 — Direct Dispatch сильно упрощает проверку потокобезопасности

**Короткий девиз 2026**:
> «Direct / Static Dispatch — это когда компилятор говорит: «Я уже всё посчитал и сделаю мгновенно».  
> final, static, private, some — ваши лучшие друзья для скорости.  
> any и @objc — только когда без них действительно нельзя.»
