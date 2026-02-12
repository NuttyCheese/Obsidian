#crash #warning #xcode #swift
### Что это значит

[[Xcode]] предупреждает, что **некоторый код никогда не будет выполнен** (unreachable).

- Это значит, что путь выполнения программы **не может дойти до этой строки кода**.
    
- Обычно это происходит из-за **раннего `return`, `break`, `continue`, `throw` или `fatalError()`**.
    
- Такие участки кода не только бесполезны, но и могут **ввести в заблуждение** при чтении кода.
    

---

### Примеры кода/сценариев возникновения

**Пример 1: Код после `return`**

```swift
func test() -> Int {
    return 5
    print("This will never be executed") // ⚠️ Will never be executed
}
```

- После `return` выполнение функции прекращается → следующая строка недостижима.
    

---

**Пример 2: Код после `throw`**

```swift
func riskyFunction() throws {
    throw NSError(domain: "", code: 1)
    print("Never runs") // ⚠️ Will never be executed
}
```

- `throw` завершает выполнение функции → последующий код unreachable.
    

---

**Пример 3: `break` или `continue` в цикле**

```swift
for i in 1...5 {
    break
    print(i) // ⚠️ Will never be executed
}
```

- После `break` итерация прекращается → следующая строка недостижима.
    

---

### Как исправить

#### 1️⃣ Удалить недостижимый код

```swift
func test() -> Int {
    return 5
    // print("This will never be executed") удаляем
}
```

---

#### 2️⃣ Переместить код до [[return]], [[throws]] или [[break]]

```swift
func test() -> Int {
    print("This will be executed")
    return 5
}
```

---

#### 3️⃣ Переписать логику, если код важен

- Если строка нужна для обработки, её надо разместить **до завершения функции** или в блоке [[do-catch]].
    

```swift
func riskyFunction() {
    do {
        try someThrowingFunction()
    } catch {
        print("Handle error")
    }
    print("This will now execute")
}
```

---

### Резюме

- Предупреждение говорит о **недостижимом коде**.
    
- Исправляется удалением или корректировкой логики.
    
- Помогает сделать код чище, предотвращает ложное впечатление, что код выполняется.
    

---
