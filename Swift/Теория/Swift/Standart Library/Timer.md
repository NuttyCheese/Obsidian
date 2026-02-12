## 1. Что такое `Timer`

**`Timer`** (ранее `NSTimer`) — это класс (`Foundation`), который позволяет:

- запускать код **через заданный интервал времени**,
    
- выполнять задачи **однократно или повторно**,
    
- интегрироваться с **[[RunLoop]]** для автоматического срабатывания в нужном потоке.
    

> Проще: `Timer` = будильник в коде, который вызывает функцию через заданное время.

---

## 2. Создание таймера

### 2.1 Повторяющийся таймер

```swift
let timer = Timer.scheduledTimer(withTimeInterval: 2.0, repeats: true) { timer in
    print("Таймер сработал")
}
```

- `withTimeInterval` — интервал в секундах ([[TimeInterval]])
    
- `repeats` — повторять ли
    
- блок/selector — что выполнять
    

### 2.2 Однократный таймер

```swift
let oneTimeTimer = Timer.scheduledTimer(withTimeInterval: 5.0, repeats: false) { _ in
    print("Таймер сработал один раз через 5 секунд")
}
```

---

## 3. Таймер с selector

```swift
let timer = Timer.scheduledTimer(timeInterval: 1.0, target: self, selector: #selector(timerFired), userInfo: nil, repeats: true)

@objc func timerFired() {
    print("Таймер сработал через selector")
}
```

- `userInfo` можно передать любые данные.
    
- Таймер автоматически добавляется в **RunLoop** главного потока.
    

---

## 4. Остановка таймера

```swift
timer.invalidate() // останавливает таймер
```

> После вызова `invalidate()` таймер больше не сработает.

---

## 5. Пример использования с отсчетом

```swift
var count = 10

let countdownTimer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { timer in
    print(count)
    count -= 1
    if count < 0 {
        timer.invalidate()
        print("Отсчет завершен")
    }
}
```

- Каждую секунду выводит число.
    
- После завершения — останавливает таймер.
    

---

## 6. Особенности

- Таймеры работают **через RunLoop**, поэтому их нужно создавать в основном потоке, если нужен UI.
    
- Использование блоков (`scheduledTimer(withTimeInterval:)`) удобнее, чем селекторов.
    
- Если не вызвать `invalidate()`, повторяющийся таймер будет жить до завершения приложения.
    
- Можно передавать **userInfo** для передачи данных в блок или selector.
    

---

## 7. Итог

- `Timer` = средство для выполнения кода через интервалы.
    
- Поддерживает **повторяющиеся и однократные** таймеры.
    
- Интегрируется с RunLoop.
    
- Управляется через `invalidate()`.
    

---
