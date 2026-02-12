## 1. Что такое `TaskLocal`

**`TaskLocal`** — это механизм, который позволяет:

- Хранить данные, **уникальные для каждой задачи ([[Task]])**
    
- Передавать эти данные между вызовами [[async]] функций **без явного аргумента**
    
- Данные автоматически наследуются дочерними задачами, если не переопределены
    

> Проще говоря: TaskLocal = «переменная, видимая только внутри текущей Task и её потомков».

---

## 2. Основные свойства

1. **Локальность** — каждая Task имеет свою копию переменной
    
2. **Наследование** — дочерние задачи получают значение из родителя, пока оно не переопределено
    
3. **Immutable внутри задачи** — значение можно изменять только через `withValue`
    

---

## 3. Создание TaskLocal

```swift
@TaskLocal
static var currentUser: String?
```

- Можно использовать как глобальную переменную, доступную только внутри Task.
    

---

## 4. Основной синтаксис использования

```swift
TaskLocal.currentUser.withValue("Alice") {
    // внутри этого блока currentUser = "Alice"
}
```

- Внутри блока значение TaskLocal доступно через `.currentValue`
    

```swift
print(TaskLocal.currentUser.currentValue) // "Alice"
```

- Вне блока значение может быть [[nil]] или значение по умолчанию
    

---

## 5. Примеры от простого к сложному

### Пример 1. Простейший TaskLocal

```swift
@TaskLocal
static var currentUser: String?

Task {
    await TaskLocal.currentUser.withValue("Alice") {
        print(TaskLocal.currentUser.currentValue ?? "No user") 
        // Output: Alice
    }
}
```

---

### Пример 2. Наследование значения в дочерней задаче

```swift
@TaskLocal
static var requestID: String?

Task {
    await TaskLocal.requestID.withValue("req123") {
        Task {
            print(TaskLocal.requestID.currentValue ?? "No request") 
            // Output: req123 (унаследовано)
        }
    }
}
```

- Дочерняя Task видит значение родителя.
    

---

### Пример 3. Переопределение значения в дочерней задаче

```swift
Task {
    await TaskLocal.requestID.withValue("parent") {
        Task {
            await TaskLocal.requestID.withValue("child") {
                print(TaskLocal.requestID.currentValue) 
                // Output: child
            }
            print(TaskLocal.requestID.currentValue) 
            // Output: parent (значение родителя)
        }
    }
}
```

---

### Пример 4. Использование в async функции

```swift
func fetchData() async {
    print("Request ID:", TaskLocal.requestID.currentValue ?? "none")
}

Task {
    await TaskLocal.requestID.withValue("req456") {
        await fetchData()
        // Output: Request ID: req456
    }
}
```

- Async функция получает доступ к TaskLocal без передачи аргумента.
    

---

### Пример 5. TaskLocal с параллельными задачами

```swift
Task {
    await TaskLocal.requestID.withValue("main") {
        await withTaskGroup(of: Void.self) { group in
            for i in 1...3 {
                group.addTask {
                    print("Task \(i):", TaskLocal.requestID.currentValue ?? "none")
                    // Все задачи видят main
                }
            }
        }
    }
}
```

- TaskLocal автоматически передаётся дочерним задачам.
    

---

## 6. Особенности TaskLocal

1. **Immutable** — изменить значение напрямую нельзя, только через `withValue`
    
2. **Наследование** — дочерние задачи наследуют значение родителя
    
3. **Безопасность потоков** — TaskLocal безопасен для многопоточности
    
4. **Можно использовать для контекста запросов, идентификаторов пользователей, логирования**
    

---

## 7. Итог

- `TaskLocal` = локальные данные для конкретной Task
    
- Доступны в async функциях и дочерних задачах без передачи аргументов
    
- Изменение только через `withValue`
    
- Отлично подходит для контекста запроса, логирования и user session
    

---
