В [[iOS]]-разработке термин "[[Delegate]]" обозначает паттерн проектирования, который позволяет объекту передавать свои события или задачи другому объекту (называемому делегатом), который обязуется реагировать на эти события или выполнить эти задачи. Паттерн делегата часто используется для обеспечения слабой связи между объектами, что способствует более гибкой и расширяемой архитектуре приложения.

**Ключевые компоненты паттерна делегата:**

1. **Delegating Object (Объект-делегат):**
    
    - Это объект, который передает свои события или задачи делегату. В [[iOS]] часто делегатом является объект, который генерирует события, например, [[UIControl]] (например, [[UIButton]]) или [[UITableViewDataSource]] в [[Swift/Теория/Swift/UIKit/TableView/UITableView]].
2. **Delegate Protocol (Протокол делегата):**
    
    - Это протокол, который определяет методы, которые делегат может реализовать. Он описывает события или задачи, которые делегат может обработать. Например, [[UITableViewDelegate]] - протокол, предоставляющий методы для управления таблицей ([[Swift/Теория/Swift/UIKit/TableView/UITableView]]).
3. **Delegate (Делегат):**
    
    - Это объект, который подписывается на протокол делегата и реализует методы, определенные в этом протоколе. Эти методы вызываются, когда происходят определенные события. В [[iOS]] это может быть объект, представляющий контроллер ([[UIViewController]]) или другой объект.

**Пример использования делегата в [[Swift]]:**

1. **Определение протокола делегата:**
```swift
protocol MyDelegate: AnyObject {
    func didSomething()
}

```
**Определение объекта-делегата:**
```swift
class DelegatingObject {
    weak var delegate: MyDelegate?
    
    func performAction() {
        // Вызываем метод делегата при выполнении действия
        delegate?.didSomething()
    }
}

```
**Реализация делегата:**
```swift
class MyDelegateImplementation: MyDelegate {
    func didSomething() {
        print("Delegate method called")
    }
}

```
**Использование:**
```swift
let delegatingObject = DelegatingObject()
let delegateImplementation = MyDelegateImplementation()

delegatingObject.delegate = delegateImplementation
delegatingObject.performAction() // Вывод: "Delegate method called"

```
В этом примере `DelegatingObject` выполняет действие и вызывает метод делегата. `MyDelegateImplementation` реализует этот метод и предоставляется в качестве делегата. Это обеспечивает слабую связь между объектами и позволяет легко заменять или добавлять новых делегатов.