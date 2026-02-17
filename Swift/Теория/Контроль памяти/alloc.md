В Swift **нет прямого аналога** метода `+alloc` из [[Objective-C]].  
Память под объекты выделяется **автоматически** при вызове инициализатора ([[init]]), а управление временем жизни объекта полностью берёт на себя **[[ARC]]** (Automatic Reference Counting).

## Краткое сравнение с Objective-C

| Действие             | Objective-C                                 | [[Swift]]                                      |
| -------------------- | ------------------------------------------- | ---------------------------------------------- |
| Выделение памяти     | `[MyClass alloc]`                           | автоматически при вызове `init`                |
| Инициализация        | `[[MyClass alloc] init]` или `initWith…`    | `MyClass()` или `MyClass(…)`                   |
| Освобождение памяти  | [[release]] / [[autorelease]] / [[dealloc]] | ARC делает это автоматически                   |
| Проверка на [[nil]]  | `if (obj == nil)`                           | `if obj == nil` или [[if let]] / [[guard let]] |
| Сильная ссылка       | [[strong]] / обычная переменная             | [[var]] / [[let]] (по умолчанию [[strong]])    |
| Слабая ссылка        | `__weak`                                    | `weak var`                                     |
| Неразрушаемая ссылка | `__unsafe_unretained`                       | [[unowned]]                                    |

## Как Swift выделяет память под классы

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
1. Выделяет память под объект (через [[Runtime]])
2. Вызывает [[init]]
3. Возвращает готовый объект

## Создание экземпляров разных типов

```swift
// 1. Класс (reference type) → куча + ARC
class Car {
    var model: String
    init(model: String) { self.model = model }
}
var car = Car(model: "Tesla Model Y")

// 2. Структура (value type) → стек или внутри объекта
struct Point {
    var x: Double
    var y: Double
}
var p1 = Point(x: 10, y: 20)     // копия значений
var p2 = p1
p2.x = 50
print(p1.x)                      // 10 — оригинал не изменился

// 3. Перечисление (value type)
enum Result {
    case success(String)
    case failure(Error)
}
var r1 = Result.success("OK")
var r2 = r1
r2 = .failure(NSError(domain: "", code: -1))
print(r1)                        // success("OK")

// 4. Массив, словарь, строка — value types с Copy-on-Write
var a1 = [1, 2, 3]
var a2 = a1                      // пока общий буфер
a2.append(4)                     // здесь происходит копирование
print(a1)                        // [1, 2, 3]
```

## ARC в действии (Automatic Reference Counting)

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

## Когда использовать [[struct]] vs [[class]]

| Нужно                               | Рекомендация   | Причина                    |
| ----------------------------------- | -------------- | -------------------------- |
| Независимые копии при присваивании  | `struct`       | Value semantics            |
| Общая изменяемая сущность           | `class`        | Reference semantics        |
| Наследование                        | `class`        | Только классы поддерживают |
| Маленькие неизменяемые данные       | `struct`       | Дешевле по памяти          |
| Объекты с жизненным циклом          | `class`        | [[deinit]], [[Observer]]   |
| Коллекции, которые часто копируются | `struct` (COW) | Эффективно                 |

**Главное правило Apple (из документации Swift)**:  
«Используйте структуры по умолчанию. Переходите на классы только тогда, когда вам действительно нужна ссылочная семантика или наследование».
