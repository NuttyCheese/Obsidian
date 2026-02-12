## 1. Что такое closure

**Closure** — это **самостоятельный блок кода**, который можно передавать и использовать как **значение**.

- Функционально похожи на функции
    
- Могут захватывать и хранить значения из внешнего контекста (**capture values**)
    
- Используются как **[[callback]]**, аргументы функций, возвращаемые значения
    

> Проще говоря: closure = «анонимная функция, которую можно передать как переменную».

---

## 2. Основные термины

| Термин                       | Описание                                                                           |
| ---------------------------- | ---------------------------------------------------------------------------------- |
| **Closure expression**       | Синтаксис для объявления closure (`{ (params) -> ReturnType in code }`)            |
| **[[Capture list]]**         | Список значений, которые closure захватывает из внешнего контекста (`[weak self]`) |
| **Trailing closure**         | Closure, переданный после скобок функции, если это последний аргумент              |
| **Escaping closure**         | Closure, который сохраняется и вызывается позже (например, в async коде)           |
| **Non-escaping closure**     | Closure вызывается сразу внутри функции                                            |
| **Shorthand argument names** | `$0`, `$1` и т.д. для краткой записи параметров                                    |

---

## 3. Основной синтаксис

```swift
let greeting = { (name: String) -> String in
    return "Hello, \(name)!"
}

print(greeting("Alice")) // Hello, Alice!
```

- Closure присваивается переменной `greeting`
    
- Можно вызывать как функцию
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простейший closure

```swift
let add = { (a: Int, b: Int) -> Int in
    return a + b
}

print(add(2, 3)) // 5
```

- Closure принимает параметры и возвращает результат
    

---

### Пример 2. Closure без параметров и возвращаемого значения

```swift
let sayHello = {
    print("Hello!")
}

sayHello() // Hello!
```

---

### Пример 3. Trailing closure в функции

```swift
func performTask(completion: () -> Void) {
    print("Task started")
    completion()
}

performTask {
    print("Task finished")
}
```

- Closure передан **после скобок функции** (trailing closure)
    

---

### Пример 4. Escaping closure (асинхронный код)

```swift
func asyncTask(completion: @escaping (String) -> Void) {
    DispatchQueue.global().async {
        completion("Async result")
    }
}

asyncTask { result in
    print(result) // Async result
}
```

- `@escaping` нужен, если closure вызывается после выхода из функции
    

---

### Пример 5. Capture values (замыкание захватывает переменные)

```swift
func makeIncrementer(amount: Int) -> () -> Int {
    var total = 0
    return {
        total += amount
        return total
    }
}

let incrementByTwo = makeIncrementer(amount: 2)
print(incrementByTwo()) // 2
print(incrementByTwo()) // 4
print(incrementByTwo()) // 6
```

- Closure захватывает `total` и `amount` из внешнего контекста
    
- `total` сохраняет своё значение между вызовами
    

---

### Пример 6. [[Weak self]] и capture list

```swift
class MyClass {
    var value = 10
    
    func doSomething() {
        DispatchQueue.global().async { [weak self] in
            guard let self = self else { return }
            print(self.value)
        }
    }
}
```

- `[weak self]` предотвращает **retain cycle** при захвате self внутри closure
    

---

## 5. Особенности closure

1. **Анонимность** — можно использовать без имени
    
2. **Capture values** — closure может хранить ссылки на внешние переменные
    
3. **Escaping vs Non-escaping** — важно для async кода
    
4. **Trailing closure** — удобная запись, когда closure последний аргумент
    
5. **Shorthand arguments** — `$0`, `$1` для краткой записи
    
6. **Reference semantics** — closure захватывает переменные как ссылки (для классов)
    

---

## 6. Итог

- **Closure** = функция без имени, которую можно присвоить переменной или передать в функцию
    
- Используется для **callback**, **[[async]] кода**, **замыканий состояния**
    
- Можно захватывать переменные и использовать `@escaping` для сохранения после выхода из функции
    
- Современный [[Swift]] активно использует closure в UI, async и коллекциях
    

---
