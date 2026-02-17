[[Xcode]] сообщает, что **функция или метод объявлены с возвращаемым значением**, но **не возвращают значение во всех возможных ветках исполнения**.

- В [[Swift]] функция с типом возвращаемого значения **обязана возвращать значение во всех путях выполнения**.
    
- Если хотя бы один путь не возвращает значение → компилятор выдаёт ошибку.
    
- Часто возникает при:
    
    1. Использовании [[if-else]], где не все ветки возвращают значение.
        
    2. Использовании [[switch]], где не все случаи покрыты.
        
    3. Пропуске `return` в конце функции.
        

---

### Примеры кода/сценариев возникновения

**Пример 1: if/else не покрывает все ветки**

```swift
func maxValue(_ a: Int, _ b: Int) -> Int {
    if a > b {
        return a
    }
    // ❌ Missing return in a function expected to return 'Int'
}
```

- Если `a <= b`, функция не возвращает значение → ошибка.
    

---

**Пример 2: switch без default**

```swift
enum Direction {
    case up, down
}

func opposite(of dir: Direction) -> Direction {
    switch dir {
    case .up:
        return .down
    }
    // ❌ Missing return in a function expected to return 'Direction'
}
```

- Не покрыт случай `.down` → компилятор требует [[return]] для всех вариантов.
    

---

**Пример 3: Функция с возвращаемым типом, но без return**

```swift
func getNumber() -> Int {
    let x = 10
    // ❌ Missing return in a function expected to return 'Int'
}
```

- Нет `return x` → ошибка.
    

---

### Как исправить

#### 1️⃣ Добавить return во все ветки

```swift
func maxValue(_ a: Int, _ b: Int) -> Int {
    if a > b {
        return a
    } else {
        return b
    }
}
```

---

#### 2️⃣ Использовать [[default]] для [[switch]]

```swift
func opposite(of dir: Direction) -> Direction {
    switch dir {
    case .up: return .down
    case .down: return .up
    }
}
```

---

#### 3️⃣ Добавить return в конце функции

```swift
func getNumber() -> Int {
    let x = 10
    return x
}
```

---

### Резюме

- Ошибка возникает, когда **функция с возвращаемым типом не возвращает значение во всех ветках исполнения**.
    
- Исправляется через:
    
    1. Добавление `return` во все ветки `if`/`else`.
        
    2. Покрытие всех случаев `switch` (или использование `default`).
        
    3. Добавление `return` в конце функции.
        
- Позволяет компилятору гарантировать, что функция всегда возвращает корректное значение и избегает runtime crash.
    

---
