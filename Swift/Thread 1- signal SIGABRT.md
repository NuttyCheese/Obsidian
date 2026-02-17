Ошибка `signal SIGABRT` указывает, что **программа была принудительно завершена системой**.

- В [[iOS]]/[[Swift]] это обычно происходит, когда вызывается функция **`abort()`** или когда **[[Xcode]] обнаруживает критическую ошибку во время выполнения**, из-за которой приложение не может продолжить работу.
    
- Чаще всего встречается при:
    
    1. Ошибках в **Interface Builder / IBOutlet**, например, подключение к несуществующему элементу.
        
    2. Force unwrap [[Optional]] (`!`) с [[nil]].
        
    3. Нарушении правил Auto Layout.
        
    4. Несовпадении Storyboard/XIB и кода.
        
    5. Проблемах с [[Core Data]] (например, некорректные fetch или save).
        
- В Xcode в консоли обычно есть **подробное сообщение об ошибке**, которое помогает понять причину:
    
    - `"unrecognized selector sent to instance"`
        
    - `"Could not load NIB in bundle"`
        
    - `"Fatal error: Unexpectedly found nil while unwrapping Optional"`
        

---

### Примеры кода/сценариев возникновения

**Пример 1: IBOutlet не подключен**

```swift
@IBOutlet weak var label: UILabel!

override func viewDidLoad() {
    super.viewDidLoad()
    label.text = "Hello" // ❌ SIGABRT, если label не подключен в Storyboard
}
```

---

**Пример 2: Force unwrap nil**

```swift
let name: String? = nil
print(name!) // ❌ SIGABRT, runtime crash
```

---

**Пример 3: Нарушение правил Auto Layout**

```swift
let view1 = UIView()
view1.translatesAutoresizingMaskIntoConstraints = false
NSLayoutConstraint.activate([
    view1.widthAnchor.constraint(equalToConstant: -10) // ❌ SIGABRT, недопустимая константа
])
```

---

**Пример 4: Несовпадение Storyboard/XIB**

```swift
let vc = UIStoryboard(name: "Main", bundle: nil)
    .instantiateViewController(withIdentifier: "NonExistentVC") // ❌ SIGABRT
```

---

### Как исправить

1. **Проверять IBOutlet и IBAction**
    
    - Убедиться, что все элементы подключены в Storyboard/XIB.
        
    - Удалить или исправить несуществующие связи.
        
2. **Безопасное развёртывание [[Optional]]**
    

```swift
if let name = name {
    print(name)
}
```

3. **Проверка Auto Layout**
    
    - Использовать допустимые константы и ограничения.
        
    - Проверять логи консоли на `Unable to satisfy constraints`.
        
4. **Проверка Storyboard/XIB идентификаторов**
    
    - Проверить правильность `Storyboard ID` и existence ViewController.
        
5. **Использовать логи Xcode**
    
    - В консоли [[Xcode]] обычно есть **точная причина SIGABRT**, которую нужно исправить.
        

---

### Резюме

- **SIGABRT** — это сигнал принудительного завершения приложения.
    
- Часто вызван критическими runtime ошибками, связанными с:
    
    - IBOutlet / IBAction
        
    - Force unwrap Optional
        
    - Auto Layout
        
    - Storyboard/XIB несоответствиями
        
- Исправляется через:
    
    1. Проверку связей в Interface Builder
        
    2. Безопасное обращение к Optional
        
    3. Правильную настройку Auto Layout
        
    4. Проверку идентификаторов Storyboard/XIB
        
- Позволяет **предотвратить падение приложения во время выполнения**.
    

---
