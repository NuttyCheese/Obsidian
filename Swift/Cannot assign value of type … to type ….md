[[Xcode]] сообщает, что вы **пытаетесь присвоить значение одного типа переменной другого типа**, и **компилятор не может выполнить автоматическое приведение типов**.

- В [[Swift]] **строгая типизация** → нельзя присвоить, например, [[Int]] переменной типа [[String]] без явного преобразования.
    
- Часто возникает при:
    
    1. Несовпадении типов переменной и значения.
        
    2. Присвоении [[Optional]] значения переменной [[non-optional]].
        
    3. Ошибках при работе с [[generic]] или [[protocol type]].
        

---

### Примеры кода/сценариев возникновения

**Пример 1: Несовпадение базовых типов**

```swift
let number: Int = 5
let text: String = number // ❌ Cannot assign value of type 'Int' to type 'String'
```

- `number` имеет тип `Int`, `text` ожидает `String` → ошибка.
    

---

**Пример 2: Optional / non-optional**

```swift
let name: String = nil // ❌ Cannot assign value of type 'NilLiteralConvertible?' to type 'String'
```

- Non-optional переменной нельзя присвоить [[nil]].
    

```swift
let optionalName: String? = nil // ✅ корректно
```

---

**Пример 3: Generic mismatch**

```swift
func process<T>(value: T) { }

let number = 10
process(value: "text") // ❌ Cannot assign value of type 'String' to type 'Int'
```

- Если generic тип уже выводится как `Int`, нельзя передавать `String`.
    

---

### Как исправить

#### 1️⃣ Привести тип явно

```swift
let text: String = String(number) // ✅ Int -> String
```

---

#### 2️⃣ Сделать переменную optional, если возможно nil

```swift
let name: String? = nil // ✅ optional тип позволяет nil
```

---

#### 3️⃣ Проверить generic типы

```swift
process(value: number) // ✅ теперь тип совпадает
```

- Или изменить generic сигнатуру, чтобы она соответствовала ожидаемому типу.
    

---

#### 4️⃣ Использовать явное преобразование между типами коллекций

```swift
let arrayInt: [Int] = [1, 2, 3]
let arrayDouble: [Double] = arrayInt.map { Double($0) } // ✅ map преобразует элементы
```

---

### Резюме

- Ошибка возникает при **несовпадении типов присваиваемого значения и переменной**.
    
- Исправляется через:
    
    1. Явное преобразование типа (`Int` → `String`, `String` → `Int` и т.д.).
        
    2. Использование optional для значений, которые могут быть nil.
        
    3. Проверку generic и protocol типов на соответствие.
        
- Помогает компилятору поддерживать строгую типизацию Swift и предотвращает [[Runtime]] ошибки.
    

---
