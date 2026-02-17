**`deinit`** — это **деинициализатор класса**, который вызывается **один раз перед удалением объекта из памяти**.

- Используется только в **классах**, т.к. [[struct]] и [[enum]] — **[[Value Type]]**, у которых нет деинициализации
    
- Позволяет **освободить ресурсы**, закрыть файлы, отменить таймеры, разорвать связи (например, [[weak]] ссылки)
    
- Не принимает параметров и не возвращает значения
    

> Проще говоря: `deinit` = «метод, который вызывается автоматически, когда объект уничтожается».

---

## 2. Основные термины

| Термин                                     | Описание                                                                       |
| ------------------------------------------ | ------------------------------------------------------------------------------ |
| **[[class]]**                              | [[Reference Type]], объект которого живёт в памяти, пока на него есть ссылки   |
| **Deinitializer**                          | Метод `deinit`, вызывается перед удалением объекта                             |
| **[[ARC]] (Automatic Reference Counting)** | Система автоматического подсчёта ссылок в [[Swift]], которая вызывает `deinit` |
| **Strong / Weak references**               | Сильные ссылки удерживают объект в памяти, слабые не удерживают                |
| **[[retain cycle]]**                      | Цикл сильных ссылок, препятствующий вызову `deinit`                            |

---

## 3. Основной синтаксис

```swift
class MyClass {
    var name: String
    init(name: String) {
        self.name = name
        print("\(name) инициализирован")
    }
    
    deinit {
        print("\(name) деинициализирован")
    }
}

var obj: MyClass? = MyClass(name: "Alice")
obj = nil // "Alice деинициализирован"
```

- `deinit` вызывается **автоматически при удалении последней сильной ссылки**
    
- Можно использовать для **очистки ресурсов или логирования**
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простой deinit

```swift
class Person {
    let name: String
    init(name: String) { self.name = name }
    deinit { print("\(name) удалён") }
}

var p: Person? = Person(name: "Bob")
p = nil // Bob удалён
```

- После присвоения [[nil]] последней ссылки объект уничтожается
    

---

### Пример 2. Deinit с ресурсами

```swift
class FileHandler {
    let fileName: String
    init(fileName: String) { self.fileName = fileName; print("Открыт файл \(fileName)") }
    deinit { print("Закрыт файл \(fileName)") }
}

var handler: FileHandler? = FileHandler(fileName: "data.txt")
handler = nil // Закрыт файл data.txt
```

- Используется для **закрытия файлов, соединений, таймеров**
    

---

### Пример 3. С retain cycle

```swift
class Node {
    var next: Node?
    deinit { print("Node удалён") }
}

var node1: Node? = Node()
var node2: Node? = Node()
node1?.next = node2
node2?.next = node1

node1 = nil
node2 = nil
// Deinit не вызывается из-за retain cycle
```

- Проблема **retain cycle** мешает вызову `deinit`
    
- Решение: использовать `weak` или `unowned`
    

---

### Пример 4. Weak ссылка для исправления retain cycle

```swift
class Node {
    weak var next: Node?
    deinit { print("Node удалён") }
}

var node1: Node? = Node()
var node2: Node? = Node()
node1?.next = node2
node2?.next = node1

node1 = nil // Node удалён
node2 = nil // Node удалён
```

- `weak` ссылка не удерживает объект в памяти → `deinit` вызывается
    

---

### Пример 5. Deinit + [[closure]]

```swift
class TimerClass {
    var timer: (() -> Void)?
    deinit { print("TimerClass удалён") }
}

var t: TimerClass? = TimerClass()
t?.timer = { [weak t] in
    print("Timer fired for \(t?.description ?? "nil")")
}
t = nil // TimerClass удалён
```

- `weak` нужен, чтобы замыкание не удерживало объект и `deinit` вызывался
    

---

## 5. Особенности deinit

1. **Вызывается автоматически при удалении последней сильной ссылки**
    
2. Используется только в **классах**
    
3. Полезен для:
    
    - Освобождения ресурсов
        
    - Отмены таймеров
        
    - Логирования жизненного цикла объектов
        
4. **Не принимает аргументы и не возвращает значения**
    
5. Если есть **retain cycle**, deinit **не вызывается**
    

---

## 6. Итог

- **deinit** = метод-деинициализатор класса
    
- Позволяет **очистить ресурсы и освободить память**
    
- Работает с ARC, вызывается автоматически при удалении последней ссылки
    
- Важно контролировать **retain cycles**, иначе `deinit` не вызовется
    

---
