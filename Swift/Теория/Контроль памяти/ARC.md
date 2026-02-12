#swift #ios #arc #automatic_reference_counting #memory_management #retain_release #reference_count #heap #object_lifecycle
# Automatic Reference Counting (ARC) в [[Swift]]

**ARC** — это **механизм автоматического управления памятью на этапе компиляции**.  
Swift **не имеет garbage collector** — вместо этого компилятор вставляет вызовы [[retain]] / [[release]] прямо в сгенерированный код.

### 1. Как работает ARC (очень просто)

```swift
class Person {
    var name: String
    deinit { print("\(name) уничтожен") }
}

var p1: Person? = Person(name: "Анна")   // retain count = 1
var p2 = p1                               // retain count = 2
p1 = nil                                  // retain count = 1
p2 = nil                                  // retain count = 0 → deinit вызван
```

### 2. Три типа ссылок

| Тип ссылки      | Синтаксис         | Увеличивает retain count? | Зануляется при dealloc? | Когда использовать                           |
| --------------- | ----------------- | ------------------------- | ----------------------- | -------------------------------------------- |
| **[[strong]]**  | [[var]] / [[let]] | Да                        | Нет                     | По умолчанию — владеет объектом              |
| **[[weak]]**    | `weak var`        | Нет                       | Да (становится [[nil]]) | Разрывает цикл, безопасно                    |
| **[[unowned]]** | `unowned let/var` | Нет                       | Нет                     | Когда уверен, что объект живёт дольше ссылки |

### 3. Классический [[retain cycle]] и его исправление

```swift
class Owner {
    var pet: Pet?
}

class Pet {
    var owner: Owner?           // цикл!
}
```

**Решение — weak или unowned**

```swift
class Pet {
    weak var owner: Owner?      // безопасно
    // или
    unowned let owner: Owner    // только если owner точно живёт дольше
}
```

### 4. Самая частая утечка — замыкания

```swift
class ViewController {
    var callback: (() -> Void)?
    
    func setup() {
        callback = { [weak self] in        // ← обязательно!
            self?.doSomething()
        }
    }
}
```

Без `[weak self]` → сильная ссылка на [[self]] → цикл → утечка.

### 5. Ключевые правила ARC

- Работает **только** с **[[reference type]]** ([[class]])
- [[struct]], [[enum]], [[tuple]] — **[[value type]]** → ARC их не касается
- Коллекции ([[Array]], [[Dictionary]], [[Set]], [[String]]) — value types с **[[Copy-on-Write]]**
- [[deinit]] вызывается **детерминированно**, сразу при RC = 0
- ARC **не ищет** и **не разрывает** циклы автоматически — это ответственность разработчика

### 6. Когда использовать weak / unowned

| Ситуация                              | Рекомендация      | Почему |
|---------------------------------------|-------------------|------|
| Делегат, datasource                   | `weak`            | Может быть nil |
| Owner → child (parent владеет)        | `weak` или `unowned` | child не должен держать owner |
| Замыкание, захватывающее self         | `[weak self]`     | Самый безопасный вариант |
| Гарантированно живёт дольше           | `unowned`         | Чуть быстрее, но опасно |

### 7. Короткий итог

- ARC — **compile-time**, **детерминированный**, **не GC**
- `strong` — владеет
- `weak` — наблюдает, зануляется
- `unowned` — доверяет, не зануляется
- Циклы — твоя ответственность
- Замыкания — главная причина утечек в iOS

**Главный принцип Swift**:
> «Используй `weak` по умолчанию. `unowned` — только когда 100% уверен в жизненном цикле».
