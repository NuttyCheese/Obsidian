## 1. Что такое `@objc`

**`@objc`** — это атрибут, который делает [[Swift]]-элемент доступным для **[[Objective-C]] [[Runtime]]**.

- Используется для **классов, методов, свойств, протоколов**
    
- Позволяет:
    
    - Вызывать Swift из Objective-C
        
    - Использовать селекторы ([[#selector]])
        
    - Подключать методы к [[@IBAction]], [[@IBOutlet]]
        
    - Работать с KVO (Key-Value Observing)
        

> Проще говоря: `@objc` = «сделай Swift-элемент доступным Objective-C».

---

## 2. Основные термины

| Термин                        | Описание                                                                               |
| ----------------------------- | -------------------------------------------------------------------------------------- |
| **Objective-C Runtime**       | Среда выполнения Objective-C, которая управляет селекторами, KVO, динамическим вызовом |
| **Selector**                  | Уникальный идентификатор метода (`#selector`)                                          |
| **IBAction / IBOutlet**       | Атрибуты для UI в Interface Builder, требуют `@objc`                                   |
| **Dynamic Dispatch**          | Вызов метода через runtime, а не напрямую                                              |
| **Optional Protocol Methods** | Методы протоколов Objective-C, которые могут быть опциональными                        |

---

## 3. Основной синтаксис

```swift
import UIKit

class MyViewController: UIViewController {
    @objc func buttonTapped() {
        print("Button tapped")
    }
}
```

- `@objc` делает метод доступным **для Objective-C runtime**
    
- Можно использовать с `#selector`
    

---

## 4. Примеры от простого к сложному

### Пример 1. Selector

```swift
let button = UIButton()
button.addTarget(self, action: #selector(buttonTapped), for: .touchUpInside)

@objc func buttonTapped() {
    print("Button pressed")
}
```

- Метод доступен через селектор
    
- Без `@objc` компилятор не даст использовать `#selector`
    

---

### Пример 2. @objc с properties

```swift
class Person: NSObject {
    @objc var name: String = "Alice"
}

let person = Person()
print(person.name)
```

- Свойство становится **доступным через Objective-C**
    

---

### Пример 3. @objc и KVO

```swift
class Person: NSObject {
    @objc dynamic var age: Int = 0
}

let person = Person()
let observation = person.observe(\.age, options: [.new]) { object, change in
    print("Age changed to \(change.newValue!)")
}

person.age = 25 // Выведет: Age changed to 25
```

- `@objc dynamic` позволяет **использовать Key-Value Observing**
    

---

### Пример 4. @objc с optional protocol methods

```swift
@objc protocol MyDelegate {
    @objc optional func didFinishTask()
}

class TaskHandler: NSObject, MyDelegate {
    func didFinishTask() {
        print("Task finished")
    }
}
```

- Протоколы Objective-C поддерживают **опциональные методы**
    
- Swift требует `@objc optional`
    

---

### Пример 5. Использование с таймером

```swift
class TimerExample: NSObject {
    var timer: Timer?

    func startTimer() {
        timer = Timer.scheduledTimer(timeInterval: 1.0, target: self, selector: #selector(timerFired), userInfo: nil, repeats: true)
    }

    @objc func timerFired() {
        print("Timer fired")
    }
}
```

- Метод таймера должен быть `@objc`, чтобы runtime смог вызвать его
    

---

## 5. Особенности `@objc`

1. Делает Swift-код доступным **для Objective-C runtime**
    
2. Используется с **методами, свойствами, протоколами**
    
3. Обязателен для **селекторов, IBAction, KVO, optional methods**
    
4. Может быть использован только с **классами наследниками NSObject**
    
5. Совмещается с `dynamic` для динамического вызова
    

---

## 6. Итог

- **@objc** = мост между Swift и Objective-C
    
- Используется для:
    
    - Селекторов (`#selector`)
        
    - IBAction / IBOutlet
        
    - KVO (`dynamic`)
        
    - Optional protocol methods
        
- Обязательно для **классов наследников NSObject**, если нужен Objective-C runtime
    
- Позволяет **динамически вызывать методы и работать с Objective-C [[API]]**
    

---
