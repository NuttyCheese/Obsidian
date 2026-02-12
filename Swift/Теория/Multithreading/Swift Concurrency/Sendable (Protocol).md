## 1. Что такое `Sendable`

**`Sendable`** — это протокол, который гарантирует, что **тип можно безопасно передавать между задачами (tasks) в [[Swift]] Concurrency**.

- Все данные в Swift Concurrency, которые передаются между [[Executor]]’ами (например, между [[actor]] или [[Task]]), должны быть **потокобезопасными**.
    
- `Sendable` говорит компилятору: «этот тип безопасно копировать и использовать в разных потоках».
    

> Проще говоря: `Sendable` = «этот объект можно передавать между задачами без гонок данных».

---

## 2. Основные правила

1. **Value types ([[struct]], [[enum]], [[tuple]])**
    
    - Если все их свойства тоже `Sendable` → автоматически `Sendable` (авто-conformance).
        
2. **[[Reference Type]] ([[class]])**
    
    - Нужно явно реализовать `Sendable` или использовать `@unchecked Sendable`.
        
    - Классы должны быть **неизменяемыми** или синхронизированными, иначе компилятор выдаст ошибку.
        
3. **Actor**
    
    - Все actor автоматически conform к `Sendable`.
        

---

## 3. Основной синтаксис

```swift
struct MyData: Sendable {
    let name: String
    let age: Int
}
```

- Здесь `MyData` — потокобезопасный тип, можно передавать между tasks.
    

---

## 4. Примеры от простого к сложному

### Пример 1. [[Value Type]] автоматически Sendable

```swift
struct Point: Sendable {
    let x: Int
    let y: Int
}

Task {
    let p = Point(x: 1, y: 2)
    print(p)
}
```

- Value types с [[let]]-свойствами автоматически conform к `Sendable`.
    

---

### Пример 2. Reference type с @unchecked Sendable

```swift
class Counter {
    var value = 0
}

extension Counter: @unchecked Sendable {}

Task {
    let counter = Counter()
    print(counter.value)
}
```

- `@unchecked Sendable` = компилятор доверяет программисту, что использование безопасно.
    

---

### Пример 3. Actor как Sendable

```swift
actor Logger {
    func log(_ message: String) { print(message) }
}

Task {
    let logger = Logger() // Logger автоматически Sendable
    await logger.log("Привет")
}
```

- Actor безопасно передавать между задачами.
    

---

### Пример 4. Использование Sendable в closure

```swift
struct DataModel: Sendable {
    let value: Int
}

Task.detached {
    let model = DataModel(value: 42)
    print(model.value) // безопасно передан в detached task
}
```

- Detached task может принимать только `Sendable` значения.
    

---

### Пример 5. Tuple и array Sendable

```swift
Task.detached {
    let numbers: [Int] = [1, 2, 3]  // [Int] автоматически Sendable
    let point: (Int, Int) = (10, 20) // tuple автоматически Sendable
    print(numbers, point)
}
```

- Все элементы массива и tuple должны быть Sendable.
    

---

## 5. Особенности

1. **Автоматическая проверка**
    
    - Компилятор проверяет, что все свойства conform к Sendable.
        
2. **@unchecked Sendable**
    
    - Используется, когда компилятор не может проверить безопасность, но программист гарантирует её.
        
    - Нужно быть осторожным: гонки данных могут возникнуть.
        
3. **Обязателен для Task.detached и async closure**
    
    - Любой объект, который передаётся в detached task, должен быть Sendable.
        
4. **Actor автоматически Sendable**
    
    - Акторы обеспечивают внутреннюю изоляцию, поэтому их можно безопасно передавать.
        

---

## 6. Итог

- `Sendable` = потокобезопасный тип, который можно передавать между задачами
    
- Структуры, enum, tuple и let свойства → автоматически Sendable
    
- Классы → нужно `@unchecked Sendable` или полностью потокобезопасная реализация
    
- Actor → автоматически Sendable
    
- Используется для detached tasks, [[async]] [[closure]] и обмена данными между executor’ами
    

---
