[[Xcode]] сообщает, что вы **передали аргумент функции/метода, который не соответствует ожидаемому типу параметра**.

- В [[Swift]] **строгая типизация аргументов функций** → нельзя передавать [[Int]] там, где ожидается [[String]], или `String?` там, где ожидается `String`.
    
- Часто ошибка возникает при:
    
    1. Передаче неправильного типа значения.
        
    2. Передаче [[Optional]] туда, где ожидается [[non-optional]].
        
    3. Несовпадении [[generic]] или [[Protocol]] типов.
        

---

### Примеры кода/сценариев возникновения

**Пример 1: Несовпадение базовых типов**

```swift
func greet(name: String) {
    print("Hello, \(name)")
}

greet(name: 42) // ❌ Cannot convert value of type 'Int' to expected argument type 'String'
```

- Аргумент `42` имеет тип `Int`, а функция ожидает `String`.
    

---

**Пример 2: Optional / non-optional**

```swift
func printName(_ name: String) {
    print(name)
}

let optionalName: String? = "Alice"
printName(optionalName) // ❌ Cannot convert value of type 'String?' to expected argument type 'String'
```

- Нужно явно извлечь значение из optional или использовать `??`:
    

```swift
printName(optionalName!) // ✅ принудительное извлечение (если уверены, что не nil)
printName(optionalName ?? "Unknown") // ✅ безопасно
```

---

**Пример 3: Generic mismatch**

```swift
func sum<T: Numeric>(_ a: T, _ b: T) -> T { a + b }

let x: Int = 5
let y: Double = 10.0

sum(x, y) // ❌ Cannot convert value of type 'Double' to expected argument type 'Int'
```

- Типы аргументов должны совпадать → можно привести:
    

```swift
sum(Double(x), y) // ✅ теперь типы совпадают
```

---

### Как исправить

#### 1️⃣ Привести тип аргумента явно

```swift
greet(name: String(42)) // ✅ Int -> String
```

---

#### 2️⃣ Для optional использовать извлечение или default

```swift
printName(optionalName ?? "Unknown") // ✅ безопасно
```

---

#### 3️⃣ Проверить generic или protocol типы

```swift
sum(Int(x), Int(y)) // ✅ приведение типов к одному generic
```

---

### Резюме

- Ошибка возникает, когда **тип передаваемого аргумента не совпадает с ожидаемым типом параметра**.
    
- Исправляется через:
    
    1. Явное приведение типов.
        
    2. Работа с optional значениям через извлечение или default.
        
    3. Проверку generic и protocol типов на соответствие.
        
- Помогает компилятору поддерживать строгую типизацию [[Swift]] и предотвращает runtime ошибки.
    

---
