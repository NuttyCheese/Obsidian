#memory_control #Swift #error 
**Цикл сильных ссылок** (retain cycle) — ситуация, когда два или более объекта **сильно** ([[strong]]) ссылаются друг на друга (или образуют замкнутую цепочку), из-за чего их **счётчик удержания** ([[retain count]]) никогда не становится равным нулю.  
В результате [[ARC]] **не может освободить** эти объекты → **утечка памяти**.

### Как возникает цикл (классический пример)

```swift
class Person {
    var apartment: Apartment?
    deinit { print("Person освобождён") }
}

class Apartment {
    var tenant: Person?
    deinit { print("Apartment освобождён") }
}

var john: Person? = Person()
var flat: Apartment? = Apartment()

john?.apartment = flat
flat?.tenant = john

john = nil
flat = nil
// → ни один deinit не вызван → утечка памяти
```

**Почему утечка?**  
- `Person` держит сильную ссылку на `Apartment` → retain count Apartment ≥ 1  
- `Apartment` держит сильную ссылку на `Person` → retain count Person ≥ 1  
- После `nil` внешних ссылок retain count остаётся > 0 → объекты живы вечно

### Как предотвратить цикл (2026 лучшие практики)

| Сценарий                                     | Рекомендация               | Почему это работает                | Когда использовать                                |
| -------------------------------------------- | -------------------------- | ---------------------------------- | ------------------------------------------------- |
| Parent → Child (владелец → зависимый)        | strong (от parent к child) | —                                  | Стандартный паттерн                               |
| Child → Parent                               | `weak var parent`          | retain count не растёт             | Самый безопасный вариант                          |
| Child → Parent (гарантированно живёт дольше) | `unowned let parent`       | быстрее, но опасно                 | [[lazy]] var после [[init]], parent владеет child |
| Замыкания внутри класса                      | `[weak self]`              | разрывает цикл                     | 99% случаев                                       |
| Замыкания с гарантией жизненного цикла       | `[unowned self]`           | быстрее, но краш при раннем deinit | Редко, только если уверен                         |

**Правильный вариант примера**:

```swift
class Apartment {
    weak var tenant: Person?      // ← ключевой момент
    deinit { print("Apartment освобождён") }
}

var john: Person? = Person()
var flat: Apartment? = Apartment()

john?.apartment = flat
flat?.tenant = john

john = nil
flat = nil
// → оба deinit вызваны
```

### Самые частые источники retain cycle в iOS 2026

1. **Замыкания** без `[weak self]`  
   ```swift
   timer = Timer.scheduledTimer(...) { self.tick() }  // цикл!
   ```

2. **Делегаты / datasource** без [[weak]]
   ```swift
   weak var delegate: SomeDelegate?   // правильно
   ```

3. **Parent ↔ Child** без слабой обратной ссылки  
   ```swift
   child.parent = self                // strong → цикл
   ```

4. **NotificationCenter** без отписки в `deinit`

### Короткий чек-лист (чтобы не было утечек)

- Все замыкания в классах → **всегда** `[weak self]`
- Делегаты, datasource, observers → **всегда** `weak var`
- Parent → Child → strong  
  Child → Parent → **weak** (или **unowned** при 100% уверенности)
- Таймеры / CADisplayLink → `[weak self]` + invalidate в `deinit`
- В [[deinit]] — лог:  
  ```swift
  deinit { print("🗑 \(type(of: self)) deinit") }
  ```
  Нет лога → retain cycle
- После навигации по экранам → **Memory Graph Debugger** + **Instruments → Leaks**

**Золотое правило Swift 2026**:
> «Внутри класса, в замыканиях и при ссылках на владельца — пиши `[weak self]` или `weak var` по умолчанию.  
> `unowned` — только если ты **точно** знаешь, что объект не умрёт раньше замыкания.»
