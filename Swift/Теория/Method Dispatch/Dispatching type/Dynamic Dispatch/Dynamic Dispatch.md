#metod_dispatch #Swift 
### 1. Что такое динамическая диспетчеризация

**Dynamic Dispatch** — это механизм, при котором выбор конкретной реализации метода (или свойства) происходит **во время выполнения программы** ([[runtime]]), а не на этапе компиляции (compile time).

Это противоположность **статической диспетчеризации** (Static Dispatch), где компилятор знает точный метод заранее и вызывает его напрямую.

**Ключевой вопрос**:  
«Какой именно метод вызвать, если у нас есть ссылка на базовый тип, а объект может быть любого подтипа?»

Ответ: **динамически**, через таблицу методов (vtable / witness table).

### 2. Три основных механизма диспетчеризации в [[Swift]]

| Механизм                                        | Когда используется                                  | Время определения | Производительность                | Примеры в коде              | Таблица методов        |
| ----------------------------------------------- | --------------------------------------------------- | ----------------- | --------------------------------- | --------------------------- | ---------------------- |
| **[[Direct Dispatch]]** (Прямая)                | `final`, `static`, методы struct/enum без протокола | Компиляция        | Максимальная (прямой вызов)       | `final func`, `static func` | Нет                    |
| **[[Table Dispatch]]** (Vtable / Witness Table) | Обычные методы классов и протоколов (не final)      | Выполнение        | Высокая (1–2 инструкции)          | `override func`, протоколы  | vtable / witness table |
| **[[Message Dispatch]]** (objc_msgSend)         | `@objc`, `dynamic`, протоколы с `@objc`             | Выполнение        | Самая медленная (поиск в таблице) | `@objc func`, `dynamic`     | objc_method_cache      |

### 3. Подробное сравнение

| Тип диспетчеризации       | Как работает                                      | Можно ли переопределить? | Можно ли в struct/enum? | Скорость (примерно) | Размер бинарника | Когда используется в 2026 |
|----------------------------|---------------------------------------------------|---------------------------|--------------------------|----------------------|------------------|----------------------------|
| Direct Dispatch            | Прямой вызов по адресу (как обычная функция)      | Нет                       | Да                       | ★★★★★                | Минимальный      | `final`, `static`, `private` методы |
| Table Dispatch (vtable)    | Через указатель на таблицу методов класса         | Да (override)             | Нет (только классы)      | ★★★★☆                | Средний          | Обычные методы классов     |
| Table Dispatch (witness)   | Через witness table протокола                     | Да                        | Да (если протокол)       | ★★★★☆                | Средний          | Протоколы без @objc        |
| Message Dispatch           | objc_msgSend — поиск в кэше/таблице               | Да                        | Нет                      | ★★☆☆☆                | Большой          | @objc, dynamic, NSObject   |

### 4. Примеры и демонстрация

#### Пример 1 — Dynamic Dispatch в классах (vtable)

```swift
class Animal {
    func makeSound() {          // динамическая диспетчеризация
        print("Generic sound")
    }
}

class Dog: Animal {
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
animals.forEach { $0.makeSound() }
// Вывод:
// Woof!
// Meow!
// Generic sound
```

**Здесь** `makeSound()` вызывается через vtable — во время выполнения определяется реальный тип объекта.

#### Пример 2 — Static Dispatch (без динамики)

```swift
class FastAnimal {
    final func makeSound() {    // final → static dispatch
        print("Fast sound")
    }
}

class FastDog: FastAnimal {
    // override func makeSound() {} // Ошибка: нельзя переопределить final
}

let fast: FastAnimal = FastDog()
fast.makeSound()  // прямой вызов, компилятор знает адрес
```

#### Пример 3 — Witness Table в протоколах (без @objc)

```swift
protocol SoundMaker {
    func makeSound()
}

struct Dog: SoundMaker {
    func makeSound() {
        print("Woof!")
    }
}

struct Cat: SoundMaker {
    func makeSound() {
        print("Meow!")
    }
}

let animals: [any SoundMaker] = [Dog(), Cat()]
animals.forEach { $0.makeSound() }
// Woof!
// Meow!
```

**Здесь** используется **witness table** — таблица соответствия протокола конкретному типу.

#### Пример 4 — Message Dispatch (@objc)

```swift
class Animal: NSObject {
    @objc dynamic func makeSound() {
        print("Generic")
    }
}

class Dog: Animal {
    override func makeSound() {
        print("Woof")
    }
}

let animal: Animal = Dog()
animal.makeSound()  // objc_msgSend → динамика
```

**objc_msgSend** — самый медленный, но позволяет runtime-магию (KVO, swizzling).

### 5. Таблица: как Swift выбирает диспетчеризацию

| Конструкция                          | Тип диспетчеризации | Почему |
|-------------------------------------|----------------------|--------|
| `final func` / `static func`        | Direct               | Нельзя переопределить |
| Обычный метод класса                | Table (vtable)       | Поддержка override |
| Метод протокола (без @objc)         | Table (witness table)| Witness table для каждого conforming типа |
| `@objc` / `dynamic` метод           | Message (objc_msgSend) | Поддержка Objective-C runtime |
| `some Protocol`                     | Direct / Table       | Конкретный тип известен компилятору |
| `any Protocol`                      | Table / Message      | Экзистенциал — динамика |

### 6. Производительность (примерные цифры 2026)

| Тип диспетчеризации | Вызов метода (нс) | Разница с direct | Где критично |
|----------------------|-------------------|-------------------|--------------|
| Direct               | ~1–2 нс           | 1×                | Горячие циклы |
| Table (vtable/witness) | ~3–5 нс         | 2–3× медленнее    | Большинство кода |
| Message (objc_msgSend) | ~10–20 нс      | 5–10× медленнее   | Редко, только @objc |

**Вывод**:  
Избегайте `any` и `@objc dynamic` в горячих путях (циклы, UI-обновления, рендеринг).

### 7. Лучшие практики 2026 года (Swift 6 strict concurrency)

- Используйте **final** и **static** везде, где возможно — максимальная скорость  
- Для протоколов — предпочитайте `some` при возврате  
- Используйте `any` только для коллекций, свойств, делегатов  
- `@objc` — только там, где нужен [[Objective-C]] [[runtime]] ([[KVO]], [[delegate]], старый [[UIKit]])  
- В [[SwiftUI]] — `some View`, `some ViewModel` — стандарт  
- В [[UIKit]] — `final class`, `private` методы, избегайте `dynamic`  
- Для максимальной производительности — используйте generics `<T: Protocol>` вместо `any`

**Короткий девиз 2026**:
> «Dynamic Dispatch — это магия полиморфизма, но за неё платят скоростью.  
> Используй `final`, `static`, `some` и generics — и твой код будет летать.  
> `any` и `@objc dynamic` — только там, где без них никак.»
