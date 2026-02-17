**Domain-Driven Design (DDD)** — это подход к проектированию и разработке сложных программных систем, который фокусируется на глубоком понимании предметной области (доменной области) и использовании этого понимания для построения архитектуры приложения. Основная идея заключается в том, чтобы модель программного обеспечения отражала бизнес-логику, а разработка велась в тесном взаимодействии с экспертами предметной области.

Если говорить о **[[Swift]]**, использование DDD возможно, несмотря на то, что язык изначально фокусируется на мобильной разработке. Подходы DDD применимы в контексте больших приложений, например, при разработке корпоративных решений, систем с микросервисной архитектурой или сложных мобильных приложений.

---

### Основные принципы DDD

1. **Уточнённое общее представление (Ubiquitous Language)**:
    
    - Все участники команды (разработчики, аналитики, заказчики) должны использовать единый язык, отражающий домен.
    - Например, если в системе есть сущности вроде "Заказ" или "Товар", то эти термины должны быть одинаковыми во всех уровнях приложения.
2. **Bounded Context (Ограниченный контекст)**:
    
    - Домен делится на контексты, каждый из которых решает свою задачу.
    - Например, контекст работы с оплатами и контекст управления заказами могут быть разными.
3. **Entity (Сущность)**:
    
    - Объект с уникальным идентификатором, изменяемым состоянием.
    - Пример: пользователь приложения.
4. **Value Object (Объект-значение)**:
    
    - Неизменяемый объект без идентификатора, который определяется своим значением.
    - Пример: адрес доставки.
5. **Aggregate (Агрегат)**:
    
    - Группа объектов, которая управляется как единое целое.
    - Пример: заказ как агрегат, включающий товары, доставку и оплату.
6. **Repository (Репозиторий)**:
    
    - Интерфейс доступа к данным агрегатов.
    - Пример: метод для получения всех заказов.

---

### Пример: Интернет-магазин на Swift с использованием DDD

#### 1. **Сущности (Entity)**

```swift
struct Product: Equatable {
    let id: UUID
    var name: String
    var price: Decimal
}
```

#### 2. **Объекты-значения (Value Object)**

```swift
struct Address: Equatable {
    let street: String
    let city: String
    let postalCode: String
}
```

#### 3. **Агрегаты (Aggregate)**

```swift
struct Order {
    let id: UUID
    var products: [Product]
    var deliveryAddress: Address
    var totalPrice: Decimal {
        products.reduce(0) { $0 + $1.price }
    }

    mutating func addProduct(_ product: Product) {
        products.append(product)
    }
}
```

#### 4. **Репозиторий (Repository)**

```swift
protocol OrderRepository {
    func save(order: Order)
    func fetchAllOrders() -> [Order]
    func findOrder(by id: UUID) -> Order?
}

class InMemoryOrderRepository: OrderRepository {
    private var orders: [UUID: Order] = [:]

    func save(order: Order) {
        orders[order.id] = order
    }

    func fetchAllOrders() -> [Order] {
        Array(orders.values)
    }

    func findOrder(by id: UUID) -> Order? {
        orders[id]
    }
}
```

#### 5. **Сервис домена (Domain Service)**

```swift
class OrderService {
    private let repository: OrderRepository

    init(repository: OrderRepository) {
        self.repository = repository
    }

    func placeOrder(products: [Product], address: Address) -> Order {
        let order = Order(id: UUID(), products: products, deliveryAddress: address)
        repository.save(order: order)
        return order
    }
}
```

#### 6. **Контроллер или приложение**

```swift
class AppController {
    private let orderService: OrderService

    init(orderService: OrderService) {
        self.orderService = orderService
    }

    func createOrder() {
        let products = [
            Product(id: UUID(), name: "Phone", price: 699.99),
            Product(id: UUID(), name: "Headphones", price: 299.99)
        ]
        let address = Address(street: "Main Street", city: "Metropolis", postalCode: "12345")
        let order = orderService.placeOrder(products: products, address: address)
        print("Order created with ID: \(order.id)")
    }
}
```

---

### Преимущества DDD в Swift

1. **Читаемость кода**: Бизнес-логика выражена так, как это понимают доменные эксперты.
2. **Гибкость**: Легче адаптироваться к изменениям в бизнес-требованиях.
3. **Тестируемость**: Логика, завязанная на агрегатах и сервисах, легко тестируется.

---

### Использование DDD в iOS

- **[[Core Data]] или [[Swift/Realm]]**: можно использовать для хранения агрегатов.
- **Combine или Swift Concurrency**: применимы для работы с асинхронными событиями.
- **Tuist или [[SPM]]**: помогает разделить домены на независимые модули.
