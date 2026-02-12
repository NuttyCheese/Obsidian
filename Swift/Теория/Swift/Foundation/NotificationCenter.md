## 1. Что такое NotificationCenter

**`NotificationCenter`** — это объект, который позволяет:

- Отправлять уведомления (notifications) от одного объекта
    
- Получать эти уведомления другими объектами, которые подписались на них
    

> Проще говоря: NotificationCenter = «почтовый сервис внутри приложения», где объекты отправляют сообщения, а подписчики их получают.

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**Notification**|Событие, которое отправляется через NotificationCenter|
|**Observer**|Подписчик, который слушает уведомления|
|**UserInfo**|Словарь `[AnyHashable: Any]?`, который передаёт дополнительные данные вместе с уведомлением|
|**name**|Имя уведомления (`Notification.Name`), по которому подписчик фильтрует уведомления|
|**object**|Объект, который отправляет уведомление (можно фильтровать по нему)|

---

## 3. Основные методы NotificationCenter

1. **addObserver** — подписка на уведомление
    

```swift
NotificationCenter.default.addObserver(
    self, 
    selector: #selector(handleNotification(_:)), 
    name: .myNotification, 
    object: nil
)
```

2. **removeObserver** — удаление подписки
    

```swift
NotificationCenter.default.removeObserver(self)
```

3. **post** — отправка уведомления
    

```swift
NotificationCenter.default.post(
    name: .myNotification, 
    object: self, 
    userInfo: ["key": "value"]
)
```

4. **Notification.Name** — тип безопасного имени уведомления
    

```swift
extension Notification.Name {
    static let myNotification = Notification.Name("myNotification")
}
```

---

## 4. Примеры от простого к сложному

### Пример 1. Простое уведомление

```swift
extension Notification.Name {
    static let testNotification = Notification.Name("testNotification")
}

class Observer {
    init() {
        NotificationCenter.default.addObserver(
            self, 
            selector: #selector(handleNotification), 
            name: .testNotification, 
            object: nil
        )
    }
    
    @objc func handleNotification(_ notification: Notification) {
        print("Notification received!")
    }
}

let observer = Observer()
NotificationCenter.default.post(name: .testNotification, object: nil)
```

- Вывод: `"Notification received!"`
    

---

### Пример 2. Уведомление с userInfo

```swift
NotificationCenter.default.post(
    name: .testNotification,
    object: nil,
    userInfo: ["username": "Alice"]
)

@objc func handleNotification(_ notification: Notification) {
    if let username = notification.userInfo?["username"] as? String {
        print("Hello, \(username)!")
    }
}
```

- Вывод: `"Hello, Alice!"`
    

---

### Пример 3. Фильтрация по объекту

```swift
let sender = NSObject()
NotificationCenter.default.post(name: .testNotification, object: sender)

NotificationCenter.default.addObserver(
    self,
    selector: #selector(handleNotification),
    name: .testNotification,
    object: sender // только уведомления от sender
)
```

- Observer получает уведомление только от указанного объекта.
    

---

### Пример 4. Использование блоков вместо selector

```swift
let token = NotificationCenter.default.addObserver(
    forName: .testNotification,
    object: nil,
    queue: .main
) { notification in
    print("Received notification via closure")
}

// Для удаления:
NotificationCenter.default.removeObserver(token)
```

- Современный способ — подписка через **closure** вместо `@objc selector`.
    

---

### Пример 5. Уведомления с многими подписчиками

```swift
let observer1 = NotificationCenter.default.addObserver(
    forName: .testNotification, object: nil, queue: .main
) { _ in print("Observer 1") }

let observer2 = NotificationCenter.default.addObserver(
    forName: .testNotification, object: nil, queue: .main
) { _ in print("Observer 2") }

NotificationCenter.default.post(name: .testNotification, object: nil)
// Вывод:
// Observer 1
// Observer 2
```

- Можно иметь **любое количество подписчиков** для одного уведомления.
    

---

## 5. Особенности NotificationCenter

1. **Singleton** — используется через `NotificationCenter.default`
    
2. **Подписка на конкретное уведомление** — фильтруется по `name` и `object`
    
3. **Поддержка userInfo** — передача данных в уведомлении
    
4. **Selector vs closure** — можно использовать `@objc func` или блоки
    
5. **Не потокобезопасно для UI** — обычно уведомления обрабатываются на главном потоке (`queue: .main`)
    

---

## 6. Итог

- `NotificationCenter` — механизм **внутри приложения для передачи сообщений между объектами**
    
- Основные шаги: **подписка → отправка → обработка**
    
- Можно передавать данные через `userInfo`
    
- Подписчиков может быть несколько, можно фильтровать по объекту
    
- Современный способ — подписка через **closure**
    

---
