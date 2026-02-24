**`weak self`** — это **слабая ссылка на объект**, которая не удерживает его в памяти.

- Используется в **замыканиях ([[closure]]), делегатах, асинхронных вызовах**
    
- Помогает **предотвратить утечки памяти**, когда объект удерживает замыкание, а замыкание — объект
    
- Значение `weak` всегда **Optional**, потому что объект может быть удалён из памяти
    

> Проще говоря: weak self = «не держи объект в памяти, чтобы его можно было удалить, если он больше не нужен».

---

## 2. Основные термины

| Термин                   | Описание                                                          |
| ------------------------ | ----------------------------------------------------------------- |
| **Strong Reference**     | Сильная ссылка, удерживает объект в памяти                        |
| **Weak Reference**       | Слабая ссылка, не удерживает объект в памяти                      |
| **[[retain cycle]]**    | Ситуация, когда два объекта удерживают друг друга → утечка памяти |
| **Closure Capture List** | Место, где можно указать `[weak self]` для слабой ссылки          |
| **Optional Self**        | [[self]] внутри `[weak self]` всегда Optional (`self?`)           |

---

## 3. Основной синтаксис

```swift
class MyClass {
    var value = 10
    func doSomething() {
        DispatchQueue.main.async { [weak self] in
            print(self?.value ?? 0)
        }
    }
}
```

- `[weak self]` указывает, что замыкание **не удерживает self**
    
- `self` внутри замыкания становится Optional → `self?`
    

---

## 4. Примеры от простого к сложному

### Пример 1. Basic weak self

```swift
class MyClass {
    var name = "Alice"
    
    func greet() {
        DispatchQueue.main.async { [weak self] in
            print(self?.name ?? "No name")
        }
    }
}

let obj = MyClass()
obj.greet()
```

- Слабая ссылка предотвращает удержание объекта `obj` замыканием
    

---

### Пример 2. Weak self с guard

```swift
class MyClass {
    var name = "Bob"
    
    func greet() {
        DispatchQueue.main.async { [weak self] in
            guard let self = self else { return }
            print(self.name)
        }
    }
}
```

- `guard let self = self` превращает Optional в **неопциональное self** внутри блока
    

---

### Пример 3. Weak self в таймере

```swift
class TimerExample {
    var count = 0
    var timer: Timer?
    
    func start() {
        timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [weak self] _ in
            self?.count += 1
            print(self?.count ?? 0)
        }
    }
}

var example: TimerExample? = TimerExample()
example?.start()
example = nil // weak self позволяет объекту освободиться
```

- Если не использовать `weak self`, таймер удерживал бы объект → retain cycle
    

---

### Пример 4. Weak self в network call

```swift
class ViewController {
    func fetchData() {
        NetworkService.loadData { [weak self] data in
            guard let self = self else { return }
            self.updateUI(data: data)
        }
    }
    
    func updateUI(data: String) {
        print("Updating UI with \(data)")
    }
}
```

- Слабая ссылка предотвращает **утечку памяти**, если VC удаляется до завершения запроса
    

---

### Пример 5. Weak self в custom closure type

```swift
class Downloader {
    var onComplete: (() -> Void)?
    
    func download() {
        onComplete = { [weak self] in
            guard let self = self else { return }
            print("Download finished for \(self)")
        }
    }
}

var downloader: Downloader? = Downloader()
downloader?.download()
downloader = nil // объект освободится благодаря weak self
```

- Полезно для **собственных замыканий и делегатов**
    

---

## 5. Особенности `weak self`

1. **Weak** = не удерживает объект в памяти
    
2. `self` становится **Optional** (`self?`)
    
3. Используется для **closure capture list** `[weak self]`
    
4. Предотвращает **retain cycles** между объектом и замыканием
    
5. В комбинации с `guard let self = self` можно работать с **неопциональным self**
    

---

## 6. Итог

- **weak self** = безопасная ссылка на self внутри замыканий
    
- Используется для **предотвращения утечек памяти**
    
- Всегда Optional → безопасно проверяем через `?` или [[guard let]]
    
- Особенно важен для:
    
    - Асинхронных вызовов ([[DispatchQueue]], таймеры)
        
    - Замыканий в свойствах классов
        
    - Сетевых запросов
        
    - Делегатов
        

---
