## 1. Что такое `isolated`

`isolated` — это **атрибут, который используется для указания того, что параметр функции или свойство [[actor]] **уже находится в изолированном контексте** актера**.

- Обычно, чтобы получить доступ к данным actor, нужно использовать [[await]].
    
- С `isolated` **вы говорите компилятору**: «этот объект уже изолирован в текущем executor, await не нужен».
    

> Проще говоря: `isolated` = «мы уже на безопасном потоке / в [[Executor]] актера, доступ к данным безопасен».

---

## 2. Где используется

1. В параметрах actor методов
    
2. В свойствах actor
    
3. В функциях, которые работают **только в контексте актера**
    

---

## 3. Основной синтаксис

```swift
actor Counter {
    var value = 0
    
    func increment(_ delta: isolated Int) {
        value += delta
    }
}
```

- Здесь `delta` помечен как `isolated`, компилятор понимает, что параметр безопасен.
    
- Можно использовать `isolated` [[self]], чтобы получить **локальный доступ к actor без [[await]]** внутри closure или метода.
    

---

## 4. Примеры

### Пример 1. Простое использование в actor

```swift
actor Counter {
    var value = 0

    func add(delta: isolated Int) {
        value += delta
    }
}

let counter = Counter()
Task {
    await counter.add(delta: 5)
}
```

- Здесь `delta` уже изолирован, await не нужен для доступа к нему.
    

---

### Пример 2. Использование `isolated self`

```swift
actor Logger {
    var messages: [String] = []

    func log(message: String) {
        Task {
            await self.addMessage(message) // обычный способ
        }
    }

    func addMessage(_ message: isolated String) {
        messages.append(message)
    }
}
```

- `isolated` позволяет функции понимать: **данные уже безопасны**, можно работать без await.
    

---

### Пример 3. Передача actor в функцию

```swift
actor Counter {
    var value = 0
}

func printValue(counter: isolated Counter) {
    // здесь мы находимся в executor Counter, можно обращаться к value напрямую
}

Task {
    let counter = Counter()
    await printValue(counter: counter)
}
```

- Вызов функции может быть безопасным без лишних await.
    

---

### Пример 4. Комбинация с MainActor

```swift
@MainActor
actor UIState {
    var labelText = ""
    
    func updateLabel(_ text: isolated String) {
        labelText = text // безопасно, await не нужен
    }
}
```

- Применимо для UI, когда мы на главном потоке, и параметр уже безопасен.
    

---

### Пример 5. Использование с [[closure]]

```swift
actor Counter {
    var value = 0
    
    func incrementWithClosure(_ closure: @Sendable (isolated Int) -> Int) {
        value += closure(value)
    }
}

let counter = Counter()
Task {
    await counter.incrementWithClosure { isolatedValue in
        return isolatedValue + 10
    }
}
```

- Closure получает **изолированный параметр**, безопасно модифицирует его.
    

---

## 5. Особенности `isolated`

1. **Работает только в actor контексте**
    
2. Позволяет **обойти await** для параметров, которые уже безопасны
    
3. Может использоваться для **self и аргументов функции**
    
4. Часто применяется вместе с **closures, делегатами и UI кодом**
    

---

## 6. Итог

- `isolated` = «уже безопасный контекст, await не нужен»
    
- Применяется к actor, его параметрам и self
    
- Упрощает доступ к данным внутри actor
    
- Особенно полезно для closures и параметров, чтобы компилятор не требовал лишнего await
    

---
