**`typealias`** — это ключевое слово, которое позволяет **давать новое имя существующему типу**.

- Не создаёт новый тип, только псевдоним
    
- Упрощает **чтение кода**, особенно с длинными типами (например, замыкания)
    
- Может использоваться для **[[struct]], [[class]], [[enum]], [[tuple]], [[closure]], [[generic]]**
    

> Проще говоря: typealias = «короткое и понятное имя для сложного или длинного типа».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**Alias**|Псевдоним для существующего типа|
|**Closure type**|Часто используется для упрощения длинных сигнатур замыканий|
|**Tuple alias**|Можно давать имя кортежу|
|**Generic alias**|Можно использовать с дженериками для читаемости|
|**No new type**|typealias не создаёт новый тип, только псевдоним|

---

## 3. Основной синтаксис

```swift
typealias Age = Int

let myAge: Age = 25
print(myAge) // 25
```

- `Age` теперь **синоним для [[Int]]**
    
- Можно использовать везде, где ожидается `Int`
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простое использование

```swift
typealias Name = String

let firstName: Name = "Alice"
let lastName: Name = "Smith"
print(firstName, lastName)
```

- Простое именование типа для читаемости
    

---

### Пример 2. Псевдоним для кортежа

```swift
typealias Point = (x: Int, y: Int)

let p1: Point = (x: 10, y: 20)
let p2: Point = (x: 5, y: 15)

func distance(from a: Point, to b: Point) -> Double {
    let dx = a.x - b.x
    let dy = a.y - b.y
    return Double(dx*dx + dy*dy).squareRoot()
}

print(distance(from: p1, to: p2)) // 18.0277...
```

- Улучшает **читаемость функций с кортежами**
    

---

### Пример 3. Псевдоним для замыкания

```swift
typealias CompletionHandler = (Bool, Error?) -> Void

func performTask(completion: CompletionHandler) {
    // имитация выполнения
    completion(true, nil)
}

performTask { success, error in
    print(success, error as Any)
}
```

- Упрощает **длинные сигнатуры замыканий**
    

---

### Пример 4. Generic typealias

```swift
typealias StringDictionary<T> = [String: T]

let dict: StringDictionary<Int> = ["a": 1, "b": 2]
print(dict) // ["a": 1, "b": 2]
```

- Полезно для **дженериков с повторяющимися шаблонами**
    

---

### Пример 5. [[Protocol]] alias

```swift
protocol Drawable { func draw() }
protocol Animatable { func animate() }

typealias AnimatableDrawable = Drawable & Animatable

struct Circle: AnimatableDrawable {
    func draw() { print("Drawing Circle") }
    func animate() { print("Animating Circle") }
}

let circle = Circle()
circle.draw()
circle.animate()
```

- Можно объединять **несколько протоколов в один alias**
    

---

## 5. Особенности typealias

1. **Создаёт псевдоним, а не новый тип**
    
2. Повышает **читаемость и сокращает длинные типы**
    
3. Может использоваться с **кортежами, замыканиями, дженериками, протоколами**
    
4. Не влияет на runtime, только на **компиляцию и синтаксис**
    

---

## 6. Итог

- **typealias** = псевдоним для существующего типа
    
- Упрощает **чтение кода**, особенно с замыканиями, кортежами, generic и протоколами
    
- Не создаёт новый тип, только имя для существующего
    
- Используется для **чистого и понятного кода**
    

---
