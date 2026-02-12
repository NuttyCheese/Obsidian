## 1. Что такое `@isolated(any)`

`@isolated(any)` — это **атрибут параметра функции**, который говорит компилятору:

👉 _"Эта функция может выполняться в изоляции того [[actor]], экземпляр которого будет передан в этот параметр"_.

То есть мы **не указываем конкретный actor**, а разрешаем функции использовать _любой actor_, который придёт в аргументе.

---

## 2. Как это работает

- Обычно если метод принимает actor, мы не можем внутри метода напрямую обращаться к его изолированным данным — нужно писать `await actor.method()`.
    
- С `@isolated(any)` мы говорим:
    
    - "Эта функция **сама будет выполняться внутри actor**, переданного как аргумент".
        
    - Это позволяет безопасно вызывать actor-методы **без [[await]]** и работать с его состоянием напрямую.
        

Можно думать так: параметр с `@isolated(any)` **переносит контекст изоляции** в функцию.

---

## 3. Простой пример

Без `@isolated(any)`:

```swift
actor Counter {
    var value = 0
    func increment() { value += 1 }
}

func doSomething(with counter: Counter) {
    // ❌ ошибка: 'counter' изолирован, нужен await
    // counter.increment()
}
```

С `@isolated(any)`:

```swift
func doSomething(with counter: isolated(any) Counter) {
    // ✅ теперь можно обращаться напрямую
    counter.increment()
}
```

---

## 4. Примеры от простого к сложному

### Пример 1. Чтение/запись из actor

```swift
actor Storage {
    var items: [String] = []

    func add(_ item: String) { items.append(item) }
}

func reset(storage: isolated(any) Storage) {
    // Мы уже внутри контекста этого storage
    storage.items.removeAll()
}
```

---

### Пример 2. Вызов без `await`

```swift
actor Logger {
    func log(_ message: String) {
        print("LOG:", message)
    }
}

func debug(logger: isolated(any) Logger) {
    logger.log("Отладка!") // ✅ без await
}
```

---

### Пример 3. Универсальная функция

```swift
func withActor<T, A: Actor>(
    _ actor: isolated(any) A,
    perform work: (isolated(any) A) -> T
) -> T {
    return work(actor)
}

actor Counter {
    var value = 0
    func increment() { value += 1 }
}

let counter = Counter()

Task {
    await withActor(counter) { counter in
        counter.increment()
        print("Value:", counter.value)
    }
}
```

---

### Пример 4. Работа с несколькими actor’ами

```swift
actor UserStorage {
    var users: [String] = []
    func add(_ user: String) { users.append(user) }
}

actor OrderStorage {
    var orders: [String] = []
    func add(_ order: String) { orders.append(order) }
}

func clearUsers(storage: isolated(any) UserStorage) {
    storage.users.removeAll()
}

func clearOrders(storage: isolated(any) OrderStorage) {
    storage.orders.removeAll()
}
```

👉 Здесь мы явно указываем разные actor, но используем одну и ту же концепцию.

---

### Пример 5. Передача изоляции в цепочке вызовов

```swift
actor Database {
    var data: [String] = []

    func add(_ item: String) { data.append(item) }
}

func addDefaultData(db: isolated(any) Database) {
    db.add("Default record")
    logData(db: db)
}

func logData(db: isolated(any) Database) {
    print("DB:", db.data)
}
```

👉 Оба метода работают **в изоляции одного и того же actor**.

---

### Пример 6. Сравнение `@isolated(any)` и [[@MainActor]]

```swift
@MainActor
func updateUI() {
    print("UI обновлён")
}

actor Service {
    func run(ui: isolated(any) @MainActor) {
        // Мы в контексте MainActor
        ui.updateUI()
    }
}
```

---

## 5. Когда использовать `@isolated(any)`

- Когда функция должна работать **внутри переданного actor** (но actor может быть любым).
    
- Чтобы писать универсальные утилиты, которые можно применять к разным actor.
    
- Чтобы не писать кучу `await`, если мы гарантированно уже "внутри" actor.
    

---

## 6. Выводы

- `@isolated(any)` — это способ **унаследовать изоляцию от аргумента-actor**.
    
- Позволяет вызывать его методы и свойства **без `await`**.
    
- Полезно для универсальных функций и снижения количества `await`.
    
- Это более современный способ работать с actor-изоляцией, чем костыли типа `@_inheritActorContext`.
    

---
