**`override`** — это ключевое слово в [[Swift]], которое **обязательно** используется при переопределении методов, свойств, сабскриптов или наблюдателей свойств, унаследованных от суперкласса.

Его главная задача — **явно указать компилятору**, что ты намеренно заменяешь реализацию из суперкласса, а не случайно создал метод с таким же именем.

### Когда `override` обязателен (и компилятор выдаст ошибку без него)

| Что переопределяем                            | Нужно ли `override`? | Пример ошибки без override                                                    | Примечание                                 |
| --------------------------------------------- | -------------------- | ----------------------------------------------------------------------------- | ------------------------------------------ |
| Метод суперкласса                             | **Да**               | Method 'move()' must be declared with 'override'                              | Самый частый случай                        |
| Хранимое свойство (var)                       | **Да**               | Property does not override any property from its superclass                   | Редко, но возможно                         |
| Вычисляемое свойство ([[var]] / [[let]])      | **Да**               | Property with type '[[Int]]' cannot override a property with a different type | Геттер/сеттер                              |
| Наблюдатель свойства ([[willSet]]/[[didSet]]) | **Да**               | Observer declared on non-observable property                                  | Только если в суперклассе есть наблюдатель |
| Сабскрипт (subscript)                         | **Да**               | Subscript must be declared with 'override'                                    | Редко используется                         |
| Ассоциированный тип в протоколе               | **Нет**              | —                                                                             | override не нужен                          |

### Самые важные правила использования `override` в 2026 году

1. **override обязателен** — если метод/свойство уже существует в любом из суперклассов (включая [[NSObject]], [[UIView]], [[UIViewController]] и т.д.)

2. **override запрещён**, если в суперклассе ничего подобного нет — компилятор выдаст ошибку:
   ```
   Method does not override any method from its superclass
   ```

3. **final** в суперклассе блокирует переопределение:
   ```swift
   class Vehicle {
       final func move() { ... }
   }
   class Car: Vehicle {
       override func move() { ... } // Ошибка: Cannot override 'final' method
   }
   ```

4. **required** + override — для обязательных методов в протоколах/классах
   ```swift
   class Base: NSObject {
       required init(coder: NSCoder) { ... }
   }
   class Child: Base {
       required init(coder: NSCoder) { ... } // required + override не пишется явно
   }
   ```

5. **override + dynamic** — для [[KVO]] и полного динамического dispatch
   ```swift
   @objc dynamic override var description: String { ... }
   ```

### Самые частые и полезные примеры 2026 года

#### 1. Переопределение методов жизненного цикла UIViewController

```swift
class ProfileViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // твоя логика после вызова суперкласса
    }
    
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        // обновить данные перед показом
    }
    
    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
        // здесь уже актуальные bounds
    }
}
```

#### 2. Переопределение computed property

```swift
class Vehicle {
    var description: String {
        "Generic vehicle"
    }
}

class Car: Vehicle {
    override var description: String {
        "Car with \(wheels) wheels"
    }
}
```

#### 3. Переопределение свойства с наблюдателем

```swift
class BaseView: UIView {
    var backgroundColor: UIColor? {
        didSet {
            print("Base color changed")
        }
    }
}

class CustomView: BaseView {
    override var backgroundColor: UIColor? {
        didSet {
            super.backgroundColor?.setFill() // вызов суперкласса
            print("Custom color changed")
        }
    }
}
```

### Короткий чек-лист «Когда писать override»

- Метод уже есть в суперклассе → **да**  
- Свойство уже есть → **да**  
- Наблюдатель свойства уже есть → **да**  
- Сабскрипт уже есть → **да**  
- Метод/свойство только в протоколе → **нет**  
- Ты придумал новое имя → **нет** (иначе ошибка)

**Короткий девиз 2026**:
> `override` — это не просто слово, это **гарантия**, что ты осознанно заменяешь поведение суперкласса.  
> Без него компилятор не даст переопределить ничего.  
> Всегда пиши `super.метод()` первым, если не хочешь сломать суперкласс.
