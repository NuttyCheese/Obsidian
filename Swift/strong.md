**`strong`** — термин, используемый в [[Swift]] и [[Objective-C]] для описания **сильной ссылки (strong reference)** на объект.  
Сильная ссылка **удерживает объект в памяти**, предотвращая его деинициализацию, пока ссылка существует.  
Относится к **Memory Management / [[ARC]] (Automatic Reference Counting)**.

---

## 🔹 Примеры кода

### 1. Простая сильная ссылка

```swift
class Person {
    var name: String
    init(name: String) { self.name = name }
}

var person1: Person? = Person(name: "Alice")
var person2 = person1 // strong reference

person1 = nil
print(person2?.name) // "Alice", объект всё ещё в памяти
```

---

### 2. Сильные ссылки в массиве

```swift
var people: [Person] = []
let person = Person(name: "Bob")
people.append(person) // массив удерживает сильную ссылку

// Объект не будет деинициализирован, пока существует массив
```

---

### 3. Проблема циклических ссылок

```swift
class Employee {
    var manager: Manager?
    deinit { print("Employee deinit") }
}

class Manager {
    var employee: Employee?
    deinit { print("Manager deinit") }
}

var emp: Employee? = Employee()
var mgr: Manager? = Manager()

emp?.manager = mgr // strong
mgr?.employee = emp // strong

emp = nil
mgr = nil
// Обе переменные не освобождаются из-за цикла strong references
```

---

### 4. Решение через [[weak]] / [[unowned]]

```swift
class Employee {
    weak var manager: Manager? // слабая ссылка
    deinit { print("Employee deinit") }
}

class Manager {
    var employee: Employee?
    deinit { print("Manager deinit") }
}

var emp: Employee? = Employee()
var mgr: Manager? = Manager()

emp?.manager = mgr
mgr?.employee = emp

emp = nil
mgr = nil
// Теперь объекты освобождаются корректно
```
