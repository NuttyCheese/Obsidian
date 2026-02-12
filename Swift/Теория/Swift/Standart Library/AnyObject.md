## 1. Что такое `AnyObject`

**`AnyObject`** — это **тип, который может представлять любой экземпляр класса ([[Reference Type]])**.

- ==В отличие от `Any`, который может хранить **любой тип**, `AnyObject` ограничен **только классами**==
    
- Используется для **работы с коллекциями объектов, [[Objective-C]] [[API]], type casting**
    
- Применяется в **protocols, arrays, dictionaries**, где нужны только объекты
    

> Проще говоря: `AnyObject` = «любой объект класса».

---

## 2. Основные термины

| Термин                          | Описание                                           |
| ------------------------------- | -------------------------------------------------- |
| **Reference type**              | Тип, который хранится по ссылке (классы)           |
| **[[Value Type]]**              | Копируется при присваивании ([[struct]], [[enum]]) |
| **Any**                         | Любой тип (value или reference)                    |
| **AnyObject**                   | Любой класс (reference type)                       |
| **Type casting (`as?`, `as!`)** | Приведение к конкретному классу                    |
| **Existential type**            | Тип, который может содержать объект любого класса  |

---

## 3. Основной синтаксис

```swift
class Person {
    var name: String
    init(name: String) { self.name = name }
}

let obj: AnyObject = Person(name: "Alice")
```

- `obj` может хранить **любой объект класса**
    
- Методы и свойства класса можно вызывать через type casting
    

---

## 4. Примеры от простого к сложному

### Пример 1. AnyObject с классом

```swift
class Animal {
    func speak() { print("Some sound") }
}

let pet: AnyObject = Animal()
(pet as! Animal).speak() // Some sound
```

- Необходим **type casting** для доступа к методам
    

---

### Пример 2. Массив AnyObject

```swift
class Dog {
    func bark() { print("Woof") }
}
class Cat {
    func meow() { print("Meow") }
}

let pets: [AnyObject] = [Dog(), Cat()]

for pet in pets {
    if let dog = pet as? Dog { dog.bark() }
    else if let cat = pet as? Cat { cat.meow() }
}
```

- Позволяет хранить **разные классы в одном массиве**
    

---

### Пример 3. AnyObject и функции

```swift
func printNames(objects: [AnyObject]) {
    for obj in objects {
        if let person = obj as? Person {
            print(person.name)
        }
    }
}

let people: [AnyObject] = [Person(name: "Alice"), Person(name: "Bob")]
printNames(objects: people) // Alice Bob
```

- Можно передавать **массив объектов разных классов**
    

---

### Пример 4. AnyObject и protocol conformance

```swift
protocol Greetable: AnyObject {
    func greet()
}

class Person: Greetable {
    func greet() { print("Hello") }
}

let greeter: AnyObject = Person()
(greeter as! Greetable).greet() // Hello
```

- Протокол с ограничением `AnyObject` может применяться только к классам
    

---

### Пример 5. AnyObject + Objective-C API

```swift
import Foundation

let array: NSMutableArray = [NSString(string: "Hello"), NSString(string: "World")]
for item in array as [AnyObject] {
    print(item)
}
```

- Часто используется для совместимости с **Objective-C коллекциями**
    

---

## 5. Особенности AnyObject

1. Ограничен **reference type** → классы
    
2. Не поддерживает **struct или [[enum]]**
    
3. Часто используется для:
    
    - Arrays/Dictionaries с объектами
        
    - Protocol с ограничением `AnyObject`
        
    - Type casting и Objective-C API
        
4. Методы и свойства объектов доступны только после **приведения к конкретному типу**
    

---

## 6. Итог

- **AnyObject** = тип, который может хранить любой объект класса
    
- Ограничен **reference types**
    
- Позволяет хранить **разные классы в коллекциях** и использовать **protocol с ограничением AnyObject**
    
- Часто применяется при **type casting и взаимодействии с Objective-C**
    

---
