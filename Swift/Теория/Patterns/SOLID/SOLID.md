#pattern #architectural_approaches 
## 📘 Определение

**SOLID** — это набор из пяти **принципов объектно-ориентированного программирования (ООП)**, направленных на **улучшение архитектуры и поддерживаемости кода**.  
Применяется в [[Swift]] и других языках с ООП.

Принципы:

1. **S** — [[Single Responsibility Principle]] (Принцип единственной ответственности)
    
2. **O** — [[Open-Closed Principle]](Принцип открытости/закрытости)
    
3. **L** — [[Liskov Substitution Principle]] (Принцип подстановки Лисков)
    
4. **I** — [[Interface Segregation Principle]] (Принцип разделения интерфейса)
    
5. **D** — [[Dependency Inversion Principle]] (Принцип инверсии зависимостей)
    

Относится к **Software Architecture / [[OOP]] / [[Swift]]**.

---

## 🔹 Примеры кода

### 1. Single Responsibility Principle

```swift
class UserFetcher {
    func fetchUser() -> String {
        return "Alice"
    }
}

class UserPrinter {
    func printUser(_ user: String) {
        print("User:", user)
    }
}

// Каждая сущность отвечает за своё
let user = UserFetcher().fetchUser()
UserPrinter().printUser(user)
```

---

### 2. Open/Closed Principle

```swift
protocol Shape {
    func area() -> Double
}

class Circle: Shape {
    var radius: Double
    init(radius: Double) { self.radius = radius }
    func area() -> Double { .pi * radius * radius }
}

class Square: Shape {
    var side: Double
    init(side: Double) { self.side = side }
    func area() -> Double { side * side }
}

// Можно добавлять новые фигуры, не изменяя существующий код
let shapes: [Shape] = [Circle(radius: 2), Square(side: 3)]
shapes.forEach { print($0.area()) }
```

---

### 3. Liskov Substitution Principle

```swift
class Bird {
    func fly() { print("Летит") }
}

class Duck: Bird {}
class Ostrich: Bird {
    override func fly() {
        // Не летаем! Нарушение LSP, лучше исключить метод fly
    }
}
```

---

### 4. Interface Segregation Principle

```swift
protocol Printer {
    func printDocument()
}

protocol Scanner {
    func scanDocument()
}

class MultiFunctionPrinter: Printer, Scanner {
    func printDocument() { print("Печать") }
    func scanDocument() { print("Сканирование") }
}
```

---

### 5. Dependency Inversion Principle

```swift
protocol Database {
    func save(data: String)
}

class SQLDatabase: Database {
    func save(data: String) { print("Сохраняем в SQL:", data) }
}

class DataManager {
    let db: Database
    init(db: Database) { self.db = db }
    func saveData(_ data: String) { db.save(data: data) }
}

let manager = DataManager(db: SQLDatabase())
manager.saveData("Тестовые данные")
```
