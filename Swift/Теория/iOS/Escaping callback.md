**Escaping callback** — это **замыкание ([[closure]]), которое может быть вызвано после того, как функция, в которую оно передано, уже завершила выполнение**.

- По умолчанию замыкания в [[Swift]] **не “выходят” за пределы функции** (non-escaping)
    
- Escaping closure объявляется с ключевым словом `@escaping`
    
- Обычно используется для **асинхронных операций**, таких как сетевые запросы, таймеры, операции с диском
    

> Проще говоря: Escaping callback = «замыкание, которое может выполняться позже, после завершения функции».

---

## 2. Основные термины

| Термин                   | Описание                                                                                                       |
| ------------------------ | -------------------------------------------------------------------------------------------------------------- |
| **Closure**              | Анонимная функция, которая может быть передана как параметр                                                    |
| **Non-escaping closure** | Замыкание, которое выполняется только внутри функции, в которую передано                                       |
| **Escaping closure**     | Замыкание, которое может выполняться после выхода из функции                                                   |
| **@escaping**            | Ключевое слово, указывающее, что closure может выйти за пределы функции                                        |
| **Capture**              | Замыкание захватывает переменные внешнего контекста (strong/weak/unowned)                                      |
| **[[retain cycle]]**    | Цикл сильных ссылок, который может возникнуть, если escaping closure захватывает self без [[weak]]/[[unowned]] |

---

## 3. Основной синтаксис

```swift
func performAsyncTask(completion: @escaping () -> Void) {
    DispatchQueue.global().async {
        // Работа в фоне
        completion()
    }
}
```

- `@escaping` → указывает, что closure будет вызван **позже**, а не внутри функции
    

---

## 4. Примеры от простого к сложному

### Пример 1. Escaping callback с асинхронным кодом

```swift
func fetchData(completion: @escaping (String) -> Void) {
    DispatchQueue.global().async {
        let data = "Hello from server"
        completion(data)
    }
}

fetchData { result in
    print(result) // Hello from server
}
```

- Замыкание вызывается **после завершения функции fetchData**
    

---

### Пример 2. Non-escaping closure

```swift
func syncTask(task: () -> Void) {
    task() // вызывается сразу
}

syncTask { print("Executed immediately") } // Executed immediately
```

- Замыкание выполняется **сразу**, не нужно `@escaping`
    

---

### Пример 3. Escaping closure + [[self]]

```swift
class ViewController {
    var name = "Alice"
    
    func loadData(completion: @escaping () -> Void) {
        DispatchQueue.global().async {
            print("Loading data for \(self.name)")
            completion()
        }
    }
}

let vc = ViewController()
vc.loadData { print("Done") }
// Output (после асинхронной работы):
// Loading data for Alice
// Done
```

- Escaping closure **захватывает self**, что может вызвать retain cycle
    

---

### Пример 4. Escaping closure + [[Weak self]]

```swift
class ViewController {
    var name = "Alice"
    
    func loadData(completion: @escaping () -> Void) {
        DispatchQueue.global().async { [weak self] in
            guard let self = self else { return }
            print("Loading data for \(self.name)")
            completion()
        }
    }
}
```

- Используем `weak self` для **предотвращения retain cycle**
    

---

### Пример 5. Escaping closure с хранением

```swift
var completionHandlers: [() -> Void] = []

func performLater(completion: @escaping () -> Void) {
    completionHandlers.append(completion) // сохраняем замыкание
}

performLater { print("Executed later") }
completionHandlers.forEach { $0() } // Executed later
```

- Escaping closure может **жить в массиве или свойстве класса**, выполняться позже
    

---

## 5. Особенности Escaping Callback

1. **Требуется @escaping**, если closure вызывается после завершения функции
    
2. **Захватывает переменные** из внешнего контекста (self, локальные переменные)
    
3. Нужно **контролировать retain cycles** через `weak` или `unowned`
    
4. Используется для **асинхронных операций, таймеров, сетевых вызовов**
    
5. Non-escaping closure **не требует @escaping**, выполняется сразу
    

---

## 6. Итог

- **Escaping callback** = замыкание, которое может выполняться позже
    
- Используется в **асинхронных операциях** и для хранения замыканий
    
- Требует **@escaping** и осторожного обращения с self
    
- Non-escaping closure = замыкание, которое выполняется сразу и безопасно
    

---
