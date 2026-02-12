`Observer` (наблюдатель) - это паттерн проектирования, который предоставляет механизм подписки и уведомления об изменениях в объекте. Он позволяет объектам (наблюдателям) реагировать на изменения в других объектах (наблюдаемых), обеспечивая слабую связь между ними.

**Основные компоненты паттерна Observer:**

1. **Subject (Наблюдаемый объект):**
    
    - Это объект, в который добавляются наблюдатели. Он содержит методы для управления подписчиками и уведомления их об изменениях. В [[iOS]] часто примерами являются объекты [[UIKit]], такие как [[NotificationCenter]] или объекты, предоставляющие KVO (Key-Value Observing).
2. **Observer (Наблюдатель):**
    
    - Это объект, который желает быть уведомленным об изменениях в наблюдаемом объекте. Обычно он реализует метод или методы, которые будут вызываться при возникновении определенных событий.

**Пример использования паттерна Observer в [[Swift]]:**

1. **Определение протокола Observer:**
```swift
protocol Observer: AnyObject {
    func update()
}

```
**Определение наблюдаемого объекта (Subject):**
```swift
class Subject {
    private var observers: [Observer] = []
    
    func addObserver(_ observer: Observer) {
        observers.append(observer)
    }
    
    func removeObserver(_ observer: Observer) {
        observers = observers.filter { $0 !== observer }
    }
    
    func notifyObservers() {
        observers.forEach { $0.update() }
    }
}

```
**Реализация наблюдателя:**
```swift
class ConcreteObserver: Observer {
    func update() {
        print("Received notification")
    }
}

```
**Использование:**
```swift
let subject = Subject()
let observer = ConcreteObserver()

subject.addObserver(observer)
subject.notifyObservers() // Вывод: "Received notification"

subject.removeObserver(observer)
subject.notifyObservers() // Нет вывода, так как наблюдатель был удален

```
В этом примере `Subject` предоставляет методы для добавления, удаления и уведомления наблюдателей. `ConcreteObserver` реализует метод `update()`, который вызывается при уведомлении.

Паттерн [[Observer]] обеспечивает гибкую систему уведомлений и позволяет объектам реагировать на изменения в других объектах без явной зависимости между ними.