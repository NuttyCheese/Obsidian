## 1. Что такое `#isolation`

`#isolation` — это **специальная конструкция/макрос**, которая возвращает **тип изоляции текущего выражения**.

- Используется **для проверки, на каком [[Executor]]’е или в каком изолированном контексте выполняется код**.
    
- Основное применение: разработка **библиотек и инструментов анализа кода**, когда нужно знать контекст изоляции функции или выражения.
    

> Проще говоря: `#isolation` говорит «компилятору: покажи мне, где этот код безопасен и в какой изоляции он находится».

---

## 2. Синтаксис

```swift
let currentIsolation = #isolation(expression)
```

- `expression` — любое выражение или вызов функции.
    
- Результат — один из видов изоляции:
    

| Вариант     | Описание                                            |
| ----------- | --------------------------------------------------- |
| [[actor]]   | выражение выполняется внутри actor                  |
| `MainActor` | выполняется в главном потоке                        |
| [[unsafe]]  | нет изоляции, глобальный executor или detached task |
| [[unknown]] | компилятор не может определить изоляцию             |

---

## 3. Примеры

### Пример 1. Проверка MainActor

```swift
@MainActor
func updateUI() {
    let iso = #isolation(self)
    print(iso) // MainActor
}

Task { @MainActor in
    await updateUI()
}
```

- Макрос `#isolation(self)` возвращает `MainActor`, потому что код выполняется в главном потоке.
    

---

### Пример 2. Actor isolation

```swift
actor Counter {
    var value = 0

    func checkIsolation() {
        let iso = #isolation(self)
        print(iso) // Counter
    }
}

let counter = Counter()
Task {
    await counter.checkIsolation()
}
```

- `#isolation(self)` = `Counter`, т.к. мы внутри [[actor]]’а.
    

---

### Пример 3. Detached task / unsafe isolation

```swift
Task.detached {
    let iso = #isolation(self)
    print(iso) // unsafe
}
```

- Detached task = глобальный executor, изоляции нет → `unsafe`.
    

---

### Пример 4. Проверка изоляции параметра

```swift
actor Counter {
    func increment(delta: isolated Int) {
        print(#isolation(delta)) // Counter
    }
}

let counter = Counter()
Task {
    await counter.increment(delta: 5)
}
```

- `delta` изолирован → макрос показывает контекст actor.
    

---

### Пример 5. Использование для анализа

```swift
@MainActor
func foo() async {
    let iso = #isolation(someAsyncFunction())
    switch iso {
    case .MainActor:
        print("Выполняется на главном потоке")
    case .actor:
        print("Внутри actor")
    case .unsafe:
        print("Глобальный executor / detached")
    default:
        print("Неизвестная изоляция")
    }
}
```

- Полезно для разработки библиотек и макросов, чтобы безопасно обрабатывать контексты.
    

---

## 4. Особенности

1. **Compile-time инструмент**
    
    - Макрос анализирует код на этапе компиляции, не влияет на [[Runtime]].
        
2. **Возвращает [[enum]]-like тип**
    
    - `.MainActor`, `.actor`, `.unsafe`, `.unknown`
        
3. **Применяется для функций, параметров, [[self]], выражений**
    
    - Можно проверять изоляцию actor, MainActor, detached task, параметров `isolated`.
        
4. **Полезно для безопасного вызова [[API]]**
    
    - Например, макрос может генерировать предупреждения при несоответствии изоляции.
        

---

## 5. Итог

- `#isolation` = expression macro, который **показывает контекст изоляции выражения**
    
- Полезен для анализа кода, проверки безопасности и генерации compile-time предупреждений
    
- Основные виды изоляции: `actor`, `MainActor`, `unsafe`, `unknown`
    
- Применим к actor, MainActor, isolated параметрам и detached задачам
    

---
