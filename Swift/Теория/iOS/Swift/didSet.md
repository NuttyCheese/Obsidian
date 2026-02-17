**`didSet`** — это **наблюдатель свойства**, который вызывается **сразу после того, как свойство получило новое значение**.

- Используется только с **хранимыми свойствами ([[var]])**
    
- Не вызывается при инициализации свойства в момент объявления, а только при последующих изменениях
    
- Можно использовать для **обновления интерфейса, валидации или логирования изменений**
    

> Проще говоря: `didSet` = «что делать после того, как свойство изменилось».

---

## 2. Основные термины

| Термин                | Описание                                                                      |
| --------------------- | ----------------------------------------------------------------------------- |
| **Property Observer** | Наблюдатель свойства: [[willSet]] (до изменения) и `didSet` (после изменения) |
| **Old Value**         | Предыдущее значение свойства, доступное через ключевое слово `oldValue`       |
| **Хранимое свойство** | Переменная, которая реально хранит данные, не вычисляемое свойство            |
| **Computed Property** | Свойство с [[Swift/Теория/Swift/Standart Library/get]]/[[Set]], не может использовать `didSet`                    |

---

## 3. Основной синтаксис

```swift
var score: Int = 0 {
    didSet {
        print("Score changed from \(oldValue) to \(score)")
    }
}

score = 10
// Output: Score changed from 0 to 10
```

- `oldValue` — встроенная переменная, содержащая предыдущее значение
    

---

## 4. Примеры от простого к сложному

### Пример 1. Логирование изменений

```swift
var temperature: Double = 20.0 {
    didSet {
        print("Temperature changed from \(oldValue) to \(temperature)")
    }
}

temperature = 25.0
// Output: Temperature changed from 20.0 to 25.0
```

- Простое отслеживание изменений значения
    

---

### Пример 2. Обновление UI через didSet

```swift
class ViewController: UIViewController {
    var labelText: String = "" {
        didSet {
            myLabel.text = labelText
        }
    }
    
    @IBOutlet weak var myLabel: UILabel!
}
```

- Когда `labelText` меняется, автоматически обновляется [[UILabel]]
    

---

### Пример 3. Валидация значения

```swift
var percentage: Int = 0 {
    didSet {
        if percentage > 100 {
            percentage = 100
        } else if percentage < 0 {
            percentage = 0
        }
        print("Percentage is now \(percentage)")
    }
}

percentage = 120
// Output: Percentage is now 100
```

- Можно автоматически **корректировать значение свойства**
    

---

### Пример 4. didSet с optional свойством

```swift
var username: String? {
    didSet {
        if let name = username {
            print("Hello, \(name)!")
        } else {
            print("Username is empty")
        }
    }
}

username = "Alice"
// Output: Hello, Alice!
username = nil
// Output: Username is empty
```

- Можно работать с **[[Optional]] значениями**
    

---

### Пример 5. didSet и willSet вместе

```swift
var score: Int = 0 {
    willSet {
        print("Score will change from \(score) to \(newValue)")
    }
    didSet {
        print("Score changed from \(oldValue) to \(score)")
    }
}

score = 50
// Output:
// Score will change from 0 to 50
// Score changed from 0 to 50
```

- `willSet` вызывается **до присваивания**, `didSet` — **после**
    

---

## 5. Особенности didSet

1. Работает **только с хранимыми свойствами (`var`)**
    
2. Не вызывается при **инициализации значения**
    
3. Доступно **oldValue** — предыдущие данные
    
4. Можно использовать вместе с **willSet** для полного контроля изменений
    
5. Часто используется для **обновления UI, логирования, валидации**
    

---

## 6. Итог

- **didSet** = наблюдатель свойства после изменения
    
- Позволяет:
    
    - Логировать изменения
        
    - Обновлять UI автоматически
        
    - Делать валидацию или коррекцию данных
        
    - Работать с Optional и обычными типами
        
- Можно комбинировать с **willSet** для контроля до и после изменения
    

---
