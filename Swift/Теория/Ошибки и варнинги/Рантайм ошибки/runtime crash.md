**Runtime crash** — это любая ошибка, которая приводит к **неожиданному завершению программы во время её выполнения**.

- Компилятор [[Swift]] не всегда может поймать эти ошибки на этапе сборки.
    
- Runtime crash может быть вызван **неправильным доступом к памяти, некорректной логикой, неверными типами или nil-значениями**.
    
- Типичные причины:
    
    1. [[Force unwrap]] [[nil]] [[Optional]] (`!`).
        
    2. Index out of range для массивов или строк.
        
    3. Attempted to read an unowned reference but object was deallocated.
        
    4. Use after free, double free или invalid memory access (редко в [[Swift]], чаще при использовании unsafe pointers).
        
    5. UI ошибки: вызов [[UIKit]] на фоне (`UI API called on a background thread`).
        

---

### Примеры кода/сценариев возникновения

**Пример 1: Force unwrap nil**

```swift
let name: String? = nil
print(name!) // ❌ runtime crash
```

---

**Пример 2: Index out of range**

```swift
let numbers = [1, 2, 3]
print(numbers[5]) // ❌ runtime crash
```

---

**Пример 3: Unowned reference**

```swift
class Person { 
    var closure: (() -> Void)?
}

var p: Person? = Person()
p?.closure = { [unowned p!] in print("Hi") }
p = nil
p?.closure?() // ❌ runtime crash
```

---

**Пример 4: UIKit на фоне**

```swift
DispatchQueue.global().async {
    label.text = "Hello" // ❌ runtime crash
}
```

---

### Как исправить

1. **Опционалы**: использовать безопасное развёртывание (`if let`, `guard let`) вместо `!`.
    
2. **Массивы и строки**: проверять границы (`indices.contains(index)`), использовать `first`, `last`.
    
3. **Unowned references**: использовать `weak` или проверку существования объекта перед доступом.
    
4. **UI операции**: выполнять на главном потоке (`DispatchQueue.main.async`).
    
5. **Общие принципы**: писать безопасный код, избегать force cast, force unwrap и unsafe pointer без крайней необходимости.
    

---

### Резюме

- **Runtime crash** — это ошибка во время выполнения, приводящая к падению приложения.
    
- Исправляется через:
    
    1. Безопасное использование Optional.
        
    2. Проверку индексов и границ.
        
    3. Контроль жизненного цикла объектов.
        
    4. Работа с UI только на главном потоке.
        
- Позволяет создавать **стабильные и безопасные приложения** на Swift.
    

---
