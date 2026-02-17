**`func`** — ключевое слово для определения **функции** в [[Swift]].

- Функция — это **блок кода, который выполняет определённую задачу**
    
- Может принимать **параметры**, возвращать **значение** и выбрасывать **ошибки**
    
- Может быть **связан с типом** (метод класса/структуры/[[enum]]) или **глобальной**
    

> Проще говоря: func = «инструкция, которую можно вызвать с аргументами и получить результат».

---

## 2. Основные термины

| Термин                    | Описание                                                                  |
| ------------------------- | ------------------------------------------------------------------------- |
| **Parameter**             | Входные данные функции (`name: String`)                                   |
| **Return type**           | Тип возвращаемого значения (`-> Int`)                                     |
| **Void**                  | Функция не возвращает значение                                            |
| **[[throws]] / rethrows** | Функция может выбрасывать ошибку                                          |
| **[[inout]]**             | Позволяет изменять переданный параметр                                    |
| **Escaping closure**      | Closure, переданный в функцию, который может сохраняться для вызова позже |
| **[[generic]] function** | Функция с параметром типа (`<T>`)                                         |
|                           |                                                                           |

---

## 3. Основной синтаксис

```swift
func greet(name: String) -> String {
    return "Hello, \(name)!"
}

let message = greet(name: "Alice")
print(message) // Hello, Alice!
```

- Функция принимает `name: String` и возвращает [[String]]
    
- Можно вызывать несколько раз с разными аргументами
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простейшая функция без параметров

```swift
func sayHello() {
    print("Hello!")
}

sayHello() // Hello!
```

- Функция без параметров и без возвращаемого значения (`Void`)
    

---

### Пример 2. Функция с параметрами и возвращаемым значением

```swift
func add(a: Int, b: Int) -> Int {
    return a + b
}

let result = add(a: 2, b: 3)
print(result) // 5
```

- Возвращает значение через [[return]]
    

---

### Пример 3. Функция с inout параметром

```swift
func increment(value: inout Int) {
    value += 1
}

var number = 10
increment(value: &number)
print(number) // 11
```

- `inout` позволяет изменять **переменную из внешнего контекста**
    

---

### Пример 4. Функция с [[throws]] и [[do-catch]]

```swift
enum MyError: Error { case failed }

func riskyFunction() throws -> String {
    if Bool.random() { return "Success" }
    else { throw MyError.failed }
}

do {
    let result = try riskyFunction()
    print(result)
} catch {
    print("Error:", error)
}
```

- Функция может **выбрасывать ошибки**
    
- Используется вместе с [[try]] и `do-catch`
    

---

### Пример 5. Функция с [[closure]] ([[completion handler]])

```swift
func performTask(completion: @escaping (String) -> Void) {
    DispatchQueue.global().async {
        completion("Task finished")
    }
}

performTask { message in
    print(message)
}
```

- Параметр функции — closure
    
- Escaping closure для асинхронного вызова
    

---

### Пример 6. [[generic]] функция

```swift
func swapValues<T>(a: inout T, b: inout T) {
    let temp = a
    a = b
    b = temp
}

var x = 1, y = 2
swapValues(a: &x, b: &y)
print(x, y) // 2 1
```

- Позволяет работать с **любыми типами** через `<T>`
    

---

### Пример 7. Функция как метод класса

```swift
class Person {
    var name: String
    init(name: String) { self.name = name }
    
    func greet() {
        print("Hello, my name is \(name)")
    }
}

let alice = Person(name: "Alice")
alice.greet() // Hello, my name is Alice
```

- Методы класса работают через **экземпляр объекта**
    

---

## 5. Особенности `func`

1. Может быть **глобальной или метод класса/структуры/enum**
    
2. Может **принимать параметры и возвращать значения**
    
3. Может использовать **inout** для изменения внешних переменных
    
4. Может **выбрасывать ошибки** (`throws`)
    
5. Может работать с **closure** и быть асинхронной
    
6. Поддерживает **generics** для универсальности
    

---

## 6. Итог

- **func** — блок кода для выполнения задачи
    
- Может иметь **параметры, возвращаемое значение, ошибки, closure и generics**
    
- Используется для **структурирования кода, повторного использования и безопасного программирования**
    

---
