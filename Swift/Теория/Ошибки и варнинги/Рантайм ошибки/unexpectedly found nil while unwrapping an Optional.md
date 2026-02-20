[[Swift]] сообщает, что **вы попытались развернуть [[Optional]] через `!`, но значение оказалось `nil`**.

- Эта ошибка **возникает во время выполнения** и является одной из самых распространённых [[Runtime]] ошибок Swift.
    
- Обычно появляется при:
    
    1. Принудительном развертывании Optional (`!`) без проверки.
        
    2. IBOutlet или IBAction, которые **не подключены** в Storyboard/XIB.
        
    3. Функции или [[API]], которые возвращают nil, а код предполагает, что значение всегда есть.
        

---

### Примеры кода/сценариев возникновения

**Пример 1: Force unwrap nil**

```swift
var name: String? = nil
print(name!) // ❌ runtime crash: unexpectedly found nil while unwrapping an Optional
```

---

**Пример 2: IBOutlet не подключен**

```swift
@IBOutlet weak var label: UILabel!

override func viewDidLoad() {
    super.viewDidLoad()
    label.text = "Hello" // ❌ если label не подключен в Storyboard → crash
}
```

---

**Пример 3: Функция возвращает nil**

```swift
func findUser(id: Int) -> String? {
    return nil
}

let user = findUser(id: 1)! // ❌ crash
```

---

### Как исправить

#### 1️⃣ Использовать безопасное развёртывание ([[if let]] / [[guard let]])

```swift
if let name = name {
    print(name)
}

guard let user = findUser(id: 1) else { return }
print(user)
```

---

#### 2️⃣ Optional chaining (`?.`)

```swift
label?.text = "Hello" // ✅ безопасно, если label = nil — ничего не произойдёт
```

---

#### 3️⃣ Default значение через `??`

```swift
let username = name ?? "Unknown"
print(username) // ✅ если name = nil → вывод "Unknown"
```

---

#### 4️⃣ Проверять IBOutlet перед использованием

```swift
if label != nil {
    label!.text = "Hello"
}
```

- Лучше использовать Optional chaining или guard-let.
    

---

### Резюме

- Ошибка возникает, когда **Optional равен nil, а вы используете `!` для развёртывания**.
    
- Исправляется через:
    
    1. Безопасное развёртывание (`if let` / `guard let`)
        
    2. Optional chaining (`?.`)
        
    3. Default значения (`??`)
        
    4. Проверку IBOutlet перед использованием
        
- Позволяет **избежать runtime crash** и безопасно работать с Optional.
    

---
