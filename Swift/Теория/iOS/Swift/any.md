**`any`** — ключевое слово [[Swift]], которое используется для **явного обозначения existential type**, то есть типа, который может содержать **любой объект, соответствующий протоколу**.

- Позволяет работать с **неопределёнными типами**, сохраняя информацию о протоколе
    
- В Swift 5.7+ использование `any` стало **явным**, раньше протоколы можно было использовать без него
    
- **Сравнение:**
    
    - `let value: Greetable` → раньше было implicit existential type
        
    - `let value: any Greetable` → современный явный синтаксис
        

> Проще говоря: `any` = «любой тип, который соответствует этому протоколу».

---

## 2. Основные термины

| Термин               | Описание                                                           |
| -------------------- | ------------------------------------------------------------------ |
| **Existential type** | Тип, который может хранить любой объект, соответствующий протоколу |
| **[[Protocol]]**     | Определяет набор требований для типа (методы, свойства)            |
| **any Protocol**     | Явный existential type                                             |
| **Dynamic dispatch** | Вызов методов через протокол во время выполнения                   |
| **Concrete type**    | Конкретная реализация протокола                                    |

---

## 3. Основной синтаксис

```swift
protocol Greetable {
    func greet()
}

struct Person: Greetable {
    func greet() { print("Hello") }
}

let someone: any Greetable = Person()
someone.greet() // Hello
```

- `any Greetable` → тип неизвестен на этапе компиляции, известны только методы протокола
    
- Методы вызываются через **dynamic dispatch**
    

---

## 4. Примеры от простого к сложному

### Пример 1. Basic any Protocol

```swift
protocol Drawable {
    func draw()
}

struct Circle: Drawable {
    func draw() { print("Drawing Circle") }
}

let shape: any Drawable = Circle()
shape.draw() // Drawing Circle
```

- `shape` может хранить **любой Drawable**
    

---

### Пример 2. Массив any Protocol

```swift
struct Square: Drawable {
    func draw() { print("Drawing Square") }
}

let shapes: [any Drawable] = [Circle(), Square()]
for shape in shapes {
    shape.draw()
}
```

- Массив может содержать **разные конкретные реализации протокола**
    

---

### Пример 3. Any с return value

```swift
func getShape(type: String) -> any Drawable {
    if type == "circle" { return Circle() }
    else { return Square() }
}

let shape = getShape(type: "circle")
shape.draw() // Drawing Circle
```

- Функция возвращает **любую реализацию протокола**
    

---

### Пример 4. Any + type casting

```swift
let shape: any Drawable = Circle()
if let circle = shape as? Circle {
    print("This is a Circle!")
}
```

- Можно привести `any Protocol` к **конкретному типу**, если нужно
    

---

### Пример 5. Any + асинхронный код

```swift
protocol AsyncTask {
    func execute() async
}

struct TaskA: AsyncTask {
    func execute() async { print("Task A executed") }
}

struct TaskB: AsyncTask {
    func execute() async { print("Task B executed") }
}

let tasks: [any AsyncTask] = [TaskA(), TaskB()]

Task {
    for task in tasks {
        await task.execute()
    }
}
```

- Работает с асинхронными протоколами, конкретная реализация неизвестна
    

---

## 5. Особенности any

1. **Existential type** → компилятору неизвестен конкретный тип
    
2. Методы вызываются через **dynamic dispatch**
    
3. Позволяет хранить **разные типы, соответствующие одному протоколу**
    
4. Можно использовать в **массивах, функциях, свойствах**
    
5. Любые операции, не объявленные в протоколе, **невозможны без type casting**
    

---

## 6. Итог

- **any Protocol** = тип, который может содержать любую реализацию протокола
    
- Позволяет писать **универсальный, гибкий код**
    
- Работает с массивами, функциями, [[async]]/[[await]]
    
- Для конкретных операций нужен **type casting**
    

---
