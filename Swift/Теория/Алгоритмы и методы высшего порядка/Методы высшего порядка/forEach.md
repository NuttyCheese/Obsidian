#algoritms #Swift #higher-order_func 
`forEach` — это метод протокола [[Sequence]], который позволяет выполнить указанное действие (замыкание) **для каждого элемента** коллекции или последовательности. Это самый простой и декларативный способ выразить «сделай что-то с каждым элементом, но не создавай новый результат».

## Основные сигнатуры

```swift
// Самая распространённая
func forEach(_ body: (Element) throws -> Void) rethrows

// С индексом (через enumerated)
sequence.enumerated().forEach { index, element in ... }

// Для словарей
dictionary.forEach { key, value in ... }
// или
dictionary.forEach { element in    // element = (key: Key, value: Value)
    ...
}
```

Важно:  
`forEach` **не возвращает** никакого значения (`Void`).  
Он предназначен исключительно для **сайд-эффектов** (печать, логирование, отправка аналитики, мутация внешнего состояния и т.п.).

## Примеры использования

### 1. Базовый проход

```swift
let names = ["Anna", "Ben", "Clara", "David"]

names.forEach { name in
    print("Привет, \(name)!")
}

// Короткая запись
names.forEach { print("Привет, \($0)!") }
```

### 2. С индексом

```swift
let scores = [85, 92, 78, 95, 88]

scores.enumerated().forEach { index, score in
    print("Игрок #\(index + 1): \(score) баллов")
}
```

### 3. Работа со словарём

```swift
let settings: [String: Any] = [
    "theme": "dark",
    "fontSize": 17,
    "notifications": true
]

settings.forEach { key, value in
    print("\(key): \(value)")
}
```

### 4. Сайд-эффекты в цепочке (очень частый паттерн)

```swift
users
    .filter { $0.isActive }
    .sorted { $0.lastActive > $1.lastActive }
    .prefix(5)
    .forEach { user in
        Analytics.track("TopActiveUserViewed", properties: ["userId": user.id])
        NotificationCenter.default.post(name: .userHighlighted, object: user)
    }
```

### 5. Мутация внешнего состояния (безопасный вариант)

```swift
var totalPrice: Decimal = 0

cart.items.forEach { item in
    totalPrice += item.price * Decimal(item.quantity)
}

print("Итого: \(totalPrice)")
```

### 6. forEach + throw (прерывание при ошибке)

```swift
do {
    try files.forEach { fileURL in
        let data = try Data(contentsOf: fileURL)
        try process(data)
    }
} catch {
    print("Ошибка при обработке файлов: \(error)")
}
```

## Важные ограничения и отличия от [[for-in]] (повторение ключевых моментов)

| Возможность                       | forEach                              | for-in                               | Рекомендация 2026 |
|-----------------------------------|--------------------------------------|--------------------------------------|-------------------|
| `break` / `continue`              | Нет (не компилируется)               | Да                                   | Если нужен ранний выход → `for-in` |
| `return` из внешней функции       | Нет (возвращает только из замыкания) | Да                                   | Только `for-in` |
| Прерывание через `throw`          | Да                                   | Да                                   | Оба подхода рабочие |
| Мутация **самой** коллекции       | Опасно, часто → краш / UB            | Опасно, но чуть проще контролировать | Избегайте в обоих случаях |
| Читаемость в функциональных цепочках | ★★★★★                                | ★★☆☆☆                                | `forEach` выигрывает |
| Производительность (релиз -O)     | Практически идентична                | Практически идентична                | Разница несущественна |

## Когда использовать forEach (современные рекомендации)

**Используйте `forEach`, если:**

- Вы пишете **функциональный стиль** и цепочку методов
- Нужны **только сайд-эффекты** (логи, аналитика, отправка уведомлений, запись в файл, мутация внешних переменных)
- Нет необходимости в `break`, `continue` или раннем `return` из функции
- Хотите подчеркнуть, что **не создаёте** новый массив / результат

**Используйте `for-in`, если:**

- Нужен контроль потока (`break`, `continue`, `return`)
- Вы работаете с индексами и хотите мутировать коллекцию (хотя лучше избегать)
- Вложенные циклы с метками (`break outer`)
- Код должен быть максимально понятен разработчикам, не привыкшим к функциональному стилю

## Антипаттерны (не делайте так в 2026)

```swift
// ❌ Плохо: forEach вместо map
let doubled = [Int]()
numbers.forEach { doubled.append($0 * 2) }     // → используйте map

// ❌ Плохо: попытка break
items.forEach {
    if condition { break }     // ← ошибка компиляции
    process($0)
}

// ❌ Опасно: мутация коллекции внутри forEach
var list = [1,2,3,4,5]
list.forEach { item in
    if item % 2 == 0 {
        list.removeAll { $0 == item }   // → почти гарантированный краш или UB
    }
}
```

## Полезные паттерны 2025–2026

```swift
// Логирование с контекстом
requestQueue.forEach { request in
    Logger.network.info("Отправка запроса", metadata: [
        "url": request.url.absoluteString,
        "method": request.httpMethod ?? "—"
    ])
}

// Асинхронные сайд-эффекты (часто с Task)
items.forEach { item in
    Task {
        await sendNotification(for: item)
    }
}

// Параллельная обработка (если порядок не важен)
items.forEach { item in
    Task.detached(priority: .background) {
        await heavyProcessing(item)
    }
}
```

## Итог — современный взгляд

`forEach` — это не просто «замена for-in с замыканием».  
Это инструмент для **декларативного выражения сайд-эффектов** в функциональном стиле.

- Хотите **трансформировать** коллекцию → [[map]] / [[compactMap]] / [[flatMap]]
- Хотите **отфильтровать** → [[filter]]
- Хотите **сделать что-то с каждым элементом без результата** → `forEach`
- Нужен **контроль потока** или **ранний выход** → `for-in`

Правильный выбор между `forEach` и `for-in` — один из признаков зрелого, идиоматичного Swift-кода в 2026 году.
