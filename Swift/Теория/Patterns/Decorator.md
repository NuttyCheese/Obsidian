Паттерн Decorator (Декоратор) - это структурный паттерн проектирования, который позволяет добавлять новое поведение или функциональность объекту, не изменяя его базовой структуры. Он достигается путем создания оберток (декораторов) вокруг базового объекта.

Основные компоненты паттерна Decorator:

1. **Component (Компонент)**: Определяет интерфейс для объектов, которые могут быть декорированы.
    
2. **ConcreteComponent (Конкретный компонент)**: Реализует интерфейс Component и представляет базовый объект, который будет декорирован.
    
3. **Decorator (Декоратор)**: Содержит ссылку на объект типа Component и реализует тот же интерфейс. Обычно абстрактный класс, который может иметь несколько конкретных реализаций.
    
4. **ConcreteDecorator (Конкретный декоратор)**: Расширяет функциональность базового объекта, добавляя новые возможности или изменяя поведение.
    

Пример использования паттерна Decorator в [[Swift]]:
```swift
// Протокол, определяющий интерфейс для объектов, которые могут быть декорированы
protocol Component {
    func operation() -> String
}

// Конкретный компонент
class ConcreteComponent: Component {
    func operation() -> String {
        return "ConcreteComponent"
    }
}

// Базовый декоратор
class Decorator: Component {
    private let component: Component
    
    init(component: Component) {
        self.component = component
    }
    
    func operation() -> String {
        return component.operation()
    }
}

// Конкретный декоратор, добавляющий дополнительную функциональность
class ConcreteDecorator: Decorator {
    override func operation() -> String {
        return "ConcreteDecorator(" + super.operation() + ")"
    }
}

// Пример использования
let component: Component = ConcreteComponent()
let decoratedComponent: Component = ConcreteDecorator(component: component)
print(decoratedComponent.operation()) // Вывод: ConcreteDecorator(ConcreteComponent)

```
Этот пример демонстрирует, как объект `ConcreteDecorator` добавляет дополнительную функциональность к базовому объекту `ConcreteComponent`, не изменяя его структуру.