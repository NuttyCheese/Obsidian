[[Xcode]] сообщает, что **компилятор не смог вывести конкретный тип для generic параметра** функции, метода или структуры.

- В [[Swift]] [[generic]] функции и типы используют **placeholder** типа (`T`, `U` и т.д.).
    
- Компилятор пытается вывести конкретный тип из **контекста вызова**.
    
- Если информация недостаточна или неоднозначна → ошибка.
    

---

### Примеры кода/сценариев возникновения

**Пример 1: Generic функция без указания типа**

```swift
func identity<T>(_ value: T) -> T {
    return value
}

let result = identity(nil) // ❌ Generic parameter 'T' could not be inferred
```

- Компилятор не может понять, какой тип `T` использовать для [[nil]].
    

---

**Пример 2: Generic коллекции**

```swift
func firstElement<T>(_ array: [T]) -> T {
    return array[0]
}

let element = firstElement([]) // ❌ Generic parameter 'T' could not be inferred
```

- Пустой массив → компилятор не может вывести тип `T`.
    

---

**Пример 3: Несоответствие типов**

```swift
func sum<T: Numeric>(_ a: T, _ b: T) -> T { a + b }

let x = 5
let y = 10.0
sum(x, y) // ❌ Generic parameter 'T' could not be inferred
```

- `x` — [[Int]], `y` — [[Double]] → компилятор не может выбрать общий `T`.
    

---

### Как исправить

#### 1️⃣ Явно указать тип generic

```swift
let result: Int = identity(0) // ✅ компилятор знает T = Int
```

```swift
let element = firstElement([1, 2, 3]) // ✅ тип Int явно выведен
```

---

#### 2️⃣ Привести аргументы к одному типу

```swift
sum(Double(x), y) // ✅ T = Double
```

---

#### 3️⃣ Добавить type annotation в вызове функции

```swift
let result = identity<String?>(nil) // ✅ явно указали T = String?
```

---

### Резюме

- Ошибка возникает, когда **Swift не может вывести тип generic параметра**.
    
- Исправляется через:
    
    1. Явное указание типа generic при вызове.
        
    2. Приведение аргументов к совместимым типам.
        
    3. Добавление type annotation для переменных.
        
- Позволяет компилятору точно сопоставлять generic параметры с реальными типами и избегать неоднозначности.
    

---
