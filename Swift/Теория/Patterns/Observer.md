Паттерн Observer (наблюдатель) - это поведенческий паттерн проектирования, который используется для оповещения об изменениях в объекте. В этом паттерне существует наблюдаемый объект (субъект) и наблюдатели, которые подписываются на изменения в нем.

Основные компоненты паттерна Observer:

1. **Subject (Субъект)**: Это объект, который содержит информацию о состоянии, изменениях или событиях. Subject также имеет список наблюдателей и методы для добавления, удаления и оповещения их об изменениях.
    
2. **Observer (Наблюдатель)**: Это объект, который подписывается на оповещения от Subject. Когда происходят изменения в Subject, он уведомляет своих наблюдателей, вызывая соответствующие методы у каждого из них.
    

Пример использования паттерна Observer в [[Swift]]:
```swift
// Протокол наблюдателя
protocol Observer: AnyObject {
    func update(data: Any)
}

// Класс субъекта
class Subject {
    private var observers = [Observer]()
    private var data: Any?
    
    func addObserver(_ observer: Observer) {
        observers.append(observer)
    }
    
    func removeObserver(_ observer: Observer) {
        observers = observers.filter { $0 !== observer }
    }
    
    func setData(_ newData: Any) {
        data = newData
        notifyObservers()
    }
    
    private func notifyObservers() {
        observers.forEach { $0.update(data: data) }
    }
}

// Класс наблюдателя
class ConcreteObserver: Observer {
    func update(data: Any) {
        print("Received data: \(data)")
    }
}

// Пример использования
let subject = Subject()
let observer1 = ConcreteObserver()
let observer2 = ConcreteObserver()

subject.addObserver(observer1)
subject.addObserver(observer2)

subject.setData("Some data") // Вывод: Received data: Some data (для обоих наблюдателей)

subject.removeObserver(observer2)

subject.setData("New data") // Вывод: Received data: New data (только для observer1)

```
В этом примере `Subject` является объектом, который содержит данные и уведомляет своих наблюдателей об изменениях. `ConcreteObserver` - это объект-наблюдатель, который реализует протокол `Observer` и реагирует на изменения, вызывая метод `update(data:)`.