**Цикл ссылок** — ситуация, когда два или более объекта **сильно** ([[strong]]) ссылаются друг на друга (или образуют замкнутую цепочку), из-за чего их **[[retain count]]** никогда не становится равным нулю.  
В результате [[ARC]] **не может освободить** эти объекты → **утечка памяти**.

### Самые частые причины циклов в [[iOS]]/Swift (2026)

| №   | Сценарий                              | Как возникает цикл                                | Частота | Как проявляется                                      |
| --- | ------------------------------------- | ------------------------------------------------- | ------- | ---------------------------------------------------- |
| 1   | Замыкание захватывает [[self]] сильно | `{ self.do() }` без `[weak self]`                 | ★★★★★   | Контроллер живёт вечно                               |
| 2   | Parent ↔ Child (двусторонняя связь)   | `parent.child = child` и `child.parent = parent`  | ★★★★☆   | Экран/объект не умирает после dismiss                |
| 3   | Делегат / dataSource сильный          | `delegate` по умолчанию strong                    | ★★★☆☆   | Контроллер держит делегат, делегат держит контроллер |
| 4   | [[NotificationCenter]] без отписки    | Не вызван `removeObserver` в `deinit`             | ★★☆☆☆   | Объект получает события после [[dealloc]]            |
| 5   | Timer / CADisplayLink сильный         | `timer = Timer.scheduledTimer…` без `[weak self]` | ★★☆☆☆   | Таймер продолжает тикать вечно                       |

### Классический пример цикла через замыкание

```swift
class ViewController {
    var timer: Timer?
    
    func startTimer() {
        timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { _ in
            self.tick()           // сильный захват self → цикл!
        }
    }
    
    func tick() {
        print("Тик")
    }
    
    deinit {
        print("Controller уничтожен")   // никогда не вызовется
    }
}
```

### Как исправить (правильные варианты)

#### Вариант 1 — самый безопасный (рекомендуется всегда)

```swift
timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [weak self] _ in
    self?.tick()
}
```

#### Вариант 2 — сокращённый синтаксис (Swift 5.3+)

```swift
timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [weak self] in
    self?.tick()
}
```

#### Вариант 3 — unowned (только когда 100% уверен)

```swift
timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [unowned self] in
    self.tick()
}
```

### Parent ↔ Child — классический цикл

```swift
class Parent {
    var child: Child?
}

class Child {
    var parent: Parent?           // сильная ссылка → цикл
}
```

**Правильный вариант**:

```swift
class Child {
    weak var parent: Parent?      // безопасно
    // или
    unowned let parent: Parent    // если parent точно живёт дольше
}
```

### Короткий чек-лист (чтобы не было утечек)

1. Все замыкания внутри классов → **всегда** `[weak self]`
2. Все делегаты, datasource, observers → **всегда** `weak var`
3. Все таймеры, CADisplayLink → `[weak self]` + invalidate в `deinit`
4. Parent → Child → strong, Child → Parent → **[[weak]]** или **[[unowned]]**
5. В `deinit` ставь лог:  
   ```swift
   deinit { print("🗑 \(type(of: self)) deinit") }
   ```
   Если лог не появляется → 99% retain cycle
6. После навигации по экранам → **Memory Graph Debugger** + **Instruments → Leaks**

### Итог — золотые правила 2026

- **По умолчанию** — `[weak self]` в замыканиях
- **unowned** — только если уверен на 100% в жизненном цикле (обычно parent → child)
- **weak** — безопасно, но требует `?` или [[guard let]]
- **Никогда** не оставляй сильный захват `self` в замыканиях внутри классов
- **Всегда** проверяй `deinit` при разработке экранов/объектов с замыканиями

**Главное правило**:
> «Если объект живёт дольше, чем должен — почти всегда это [[retain cycle]] через замыкание или делегат.  
> Пиши `[weak self]` — это бесплатно и спасает от 95% утечек.»
