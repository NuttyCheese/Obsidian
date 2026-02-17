**`final`** — модификатор в [[Swift]], который запрещает **наследование класса** или **переопределение метода/свойства**.  
Используется для оптимизации (компилятор может применить [[Compile-time Dispatch]]) и для защиты архитектуры от изменения наследниками.

---

## 🔹 Примеры кода

### 1. `final` класс

```swift
final class Animal {
    func speak() {
        print("Животное издает звук")
    }
}

// class Dog: Animal {} // ❌ Ошибка: наследование запрещено
```

---

### 2. `final` метод в классе

```swift
class Animal {
    final func eat() {
        print("Животное ест")
    }
}

class Dog: Animal {
    // override eat() {} // ❌ Ошибка: переопределение запрещено
}
```

---

### 3. `final` свойство

```swift
class Car {
    final var brand = "Toyota"
}

class SportCar: Car {
    // override var brand = "Ferrari" // ❌ Ошибка
}
```

---

### 4. `final` с методами и compile-time dispatch

```swift
final class Printer {
    func printMessage() {
        print("Сообщение")
    }
}

let printer: Printer = Printer()
printer.printMessage() // вызов известен на этапе компиляции
```

---

### 5. `final` для оптимизации структуры

```swift
struct Circle {
    let radius: Double
    // Структуры всегда final по сути — нельзя наследовать
}
```
