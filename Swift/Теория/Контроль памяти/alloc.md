#memory #arc #alloc #objective-c #swift #init #deinit #heap #stack

---
### Определение
В Swift **нет прямого аналога** метода `+alloc` из [[Objective-C]]. Память под объекты выделяется **автоматически** при вызове инициализатора ([[init]]), а управление временем жизни объекта полностью берёт на себя **[[ARC]]** (Automatic Reference Counting) .

В Objective-C процесс создания объекта был двухфазным: сначала выделение памяти (`alloc`), затем инициализация (`init`). В [[Swift]] это объединено в единую операцию вызова инициализатора, что делает код более безопасным и менее подверженным ошибкам.

### Зачем это знать iOS-разработчику?
1.  **Понимание управления памятью:** Важно знать, как Swift управляет памятью, чтобы избегать утечек и циклов сильных ссылок.
2.  **Выбор между class и struct:** Понимание семантики ссылочных и значимых типов критически важно для производительности и корректности кода.
3.  **Работа с legacy Objective-C кодом:** При поддержке старых проектов или использовании Objective-C библиотек нужно понимать различия.
4.  **Оптимизация производительности:** Знание того, как выделяется память, помогает писать эффективный код.

---

### Сравнение: Objective-C vs Swift

| Действие | Objective-C | Swift |
|----------|-------------|-------|
| **Выделение памяти** | `[MyClass alloc]` | автоматически при вызове `init` |
| **Инициализация** | `[[MyClass alloc] init]` или `initWith…` | `MyClass()` или `MyClass(…)` |
| **Освобождение памяти** | `release` / `autorelease` / `dealloc` | ARC делает это автоматически |
| **Проверка на nil** | `if (obj == nil)` | `if obj == nil` или `if let` / `guard let` |
| **Сильная ссылка** | `strong` / обычная переменная | `var` / `let` (по умолчанию `strong`) |
| **Слабая ссылка** | `__weak` | `weak var` |
| **Неразрушаемая ссылка** | `__unsafe_unretained` | `unowned` |

### Как Swift выделяет память под классы

```swift
class Person {
    let name: String
    
    init(name: String) {
        self.name = name
        print("Person \(name) создан")
    }
    
    deinit {
        print("Person \(name) уничтожен")
    }
}

// Создание экземпляра
var person: Person? = Person(name: "Анна")   // → выделяется память + вызывается init
person = nil                                  // → ARC видит, что ссылок больше нет → deinit
// → Person Анна уничтожен
```

**Ключевой момент**:  
`Person(name: "Анна")` — это **одна атомарная операция**, которая:
1.  Выделяет память под объект (через Swift Runtime).
2.  Вызывает `init`.
3.  Возвращает готовый объект.

---

### Создание экземпляров разных типов

#### 1. Класс ([[reference type]]) — память в куче + ARC

```swift
class Car {
    var model: String
    init(model: String) { self.model = model }
}

var car1 = Car(model: "Tesla Model Y")
var car2 = car1  // обе переменные ссылаются на один и тот же объект
car2.model = "Tesla Model 3"
print(car1.model)  // "Tesla Model 3" — объект изменился
```

#### 2. Структура ([[value type]]) — память на стеке или внутри объекта

```swift
struct Point {
    var x: Double
    var y: Double
}

var p1 = Point(x: 10, y: 20)  // копия значений
var p2 = p1                    // создается независимая копия
p2.x = 50
print(p1.x)                     // 10 — оригинал не изменился
```

#### 3. Перечисление (value type)

```swift
enum Result {
    case success(String)
    case failure(Error)
}

var r1 = Result.success("OK")
var r2 = r1                      // копия
r2 = .failure(NSError(domain: "", code: -1))
print(r1)                        // success("OK")
```

#### 4. Массив, словарь, строка — value types с [[Copy-on-Write]]

```swift
var a1 = [1, 2, 3]
var a2 = a1                      // пока общий буфер (COW)
a2.append(4)                      // здесь происходит копирование
print(a1)                         // [1, 2, 3]
```

---
### [[ARC]] в действии (Automatic Reference Counting)

```swift
class Node {
    var value: Int
    weak var parent: Node?      // weak — предотвращает retain cycle
    init(value: Int) { self.value = value }
    deinit { print("Node \(value) уничтожен") }
}

var root: Node? = Node(value: 0)
var child: Node? = Node(value: 1)

root?.parent = child
child?.parent = root            // цикл сильных ссылок

root = nil
child = nil                     // без weak → утечка памяти
// с weak → deinit вызывается у обоих
```

### Правила работы с weak и unowned

| Ситуация                                        | Решение                            | Пример                                    |
| ----------------------------------------------- | ---------------------------------- | ----------------------------------------- |
| Ссылка может стать [[nil]]                      | [[weak]]                           | [[delegate]], parent                      |
| Ссылка никогда не будет nil после инициализации | [[unowned]]                        | [[self]] в замыкании (если гарантировано) |
| Замыкание, захватывающее self                   | `[weak self]` или `[unowned self]` | Обработчики событий                       |

```swift
class DataLoader {
    var onComplete: (() -> Void)?
    
    func load() {
        onComplete = { [weak self] in
            guard let self = self else { return }
            self.processData()
        }
    }
    
    func processData() { }
}
```

---

### Когда использовать struct vs class

| Нужно                               | Рекомендация   | Причина                                 |
| ----------------------------------- | -------------- | --------------------------------------- |
| Независимые копии при присваивании  | [[struct]]     | Value semantics                         |
| Общая изменяемая сущность           | [[class]]      | Reference semantics                     |
| Наследование                        | `class`        | Только классы поддерживают наследование |
| Маленькие неизменяемые данные       | `struct`       | Дешевле по памяти                       |
| Объекты с жизненным циклом          | `class`        | `deinit`, наблюдатели                   |
| Коллекции, которые часто копируются | `struct` (COW) | Эффективно                              |

**Главное правило Apple (из документации Swift)**:  
«Используйте структуры по умолчанию. Переходите на классы только тогда, когда вам действительно нужна ссылочная семантика или наследование» .

---

### Пример: Смешанное использование struct и class

```swift
// Структура для неизменяемых данных
struct Address {
    let street: String
    let city: String
    let zipCode: String
}

// Класс для изменяемой сущности с идентичностью
class User {
    let id: UUID
    var name: String
    var address: Address
    
    init(id: UUID, name: String, address: Address) {
        self.id = id
        self.name = name
        self.address = address
    }
}

// Использование
let address = Address(street: "Main St", city: "Boston", zipCode: "02101")
let user = User(id: UUID(), name: "Alice", address: address)

// Копирование структуры
var addressCopy = address
addressCopy.city = "New York"  // меняется только копия

// Ссылка на класс
let user2 = user
user2.name = "Bob"              // меняется оригинал
print(user.name)                // "Bob"
```

---

### Продвинутое управление памятью

#### 1. **[[Autoreleasepool]]**
Для управления памятью в циклах с большим количеством временных объектов.

```swift
func processLargeArray(_ items: [String]) {
    for i in 0..<items.count {
        autoreleasepool {
            // Временные объекты создаются и освобождаются на каждой итерации
            let temp = items[i].uppercased()
            print(temp)
        }
    }
}
```

#### 2. **withExtendedLifetime**
Гарантирует, что объект не будет освобожден до выполнения блока.

```swift
func unsafeOperation() {
    let resource = SomeResource()
    withExtendedLifetime(resource) {
        // Используем resource здесь
        // после выхода из блока resource может быть освобожден
    }
}
```

#### 3. **[[unsafeBitCast]]**
Для низкоуровневой работы с памятью (только при крайней необходимости).

```swift
let x: Int = 42
let y = unsafeBitCast(x, to: Double.self)  // преобразование без проверок типов
```

### Итог
Swift предлагает современную и безопасную модель управления памятью, где:
- **Классы** управляются ARC и хранятся в куче.
- **Структуры и перечисления** — value types, хранятся на стеке (или внутри объектов).
- **Copy-on-Write** оптимизирует работу с коллекциями.
- **Weak и unowned** предотвращают циклы сильных ссылок.
- **Нет прямого аналога `alloc`** — выделение памяти происходит автоматически при инициализации .

Понимание этих механизмов необходимо для написания эффективного и безопасного кода без утечек памяти .