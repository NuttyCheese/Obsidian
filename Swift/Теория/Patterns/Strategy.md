В разработке на языке [[Swift]] **Strategy** (стратегия) - это поведенческий паттерн проектирования, который позволяет определить семейство алгоритмов, инкапсулировать каждый из них и сделать их взаимозаменяемыми. Паттерн Strategy позволяет выбирать алгоритм динамически в зависимости от контекста.

Основные участники паттерна:

1. **Context (Контекст)**: Это объект, который содержит ссылку на объект стратегии и делегирует ему выполнение работы.
    
2. **Strategy (Стратегия)**: Это протокол или абстрактный класс, определяющий общий интерфейс для всех конкретных стратегий.
    
3. **Concrete Strategies (Конкретные стратегии)**: Это классы, реализующие конкретные алгоритмы, определенные в протоколе Strategy.
    

Пример реализации паттерна Strategy в [[Swift]]:
```swift
// Протокол определяющий интерфейс стратегии
protocol Strategy {
    func execute()
}

// Конкретные стратегии реализующие различные алгоритмы
class ConcreteStrategyA: Strategy {
    func execute() {
        print("Executing strategy A")
    }
}

class ConcreteStrategyB: Strategy {
    func execute() {
        print("Executing strategy B")
    }
}

// Контекст, который использует стратегии
class Context {
    private var strategy: Strategy
    
    init(strategy: Strategy) {
        self.strategy = strategy
    }
    
    func setStrategy(strategy: Strategy) {
        self.strategy = strategy
    }
    
    func executeStrategy() {
        strategy.execute()
    }
}

// Использование
let context = Context(strategy: ConcreteStrategyA())
context.executeStrategy() // Вывод: Executing strategy A

context.setStrategy(strategy: ConcreteStrategyB())
context.executeStrategy() // Вывод: Executing strategy B

```
В этом примере `Context` представляет объект, который использует стратегии. Он имеет методы для установки стратегии и выполнения работы с текущей стратегией. Каждая конкретная стратегия (например, `ConcreteStrategyA` и `ConcreteStrategyB`) реализует метод `execute()`, который выполняет определенный алгоритм. При необходимости контекст может изменять стратегию динамически, не зная подробностей реализации каждой стратегии.