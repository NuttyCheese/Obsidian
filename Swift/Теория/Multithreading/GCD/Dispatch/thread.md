## 1. Что такое `Thread`

**`Thread`** — это **отдельный поток выполнения кода**.

- Позволяет запускать задачи **параллельно** основному потоку (UI).
    
- Каждый поток имеет собственный **[[RunLoop]]**, стек и регистры.
    
- На [[iOS]] чаще используют **GCD ([[DispatchQueue]])** или **OperationQueue**, но `Thread` все еще доступен для низкоуровневого управления.
    

> Проще: `Thread` = отдельная «нитка», на которой может выполняться код параллельно с главным потоком.

---

## 2. Создание и запуск потока

### 2.1 Инициализация через замыкание

```swift
let thread = Thread {
    for i in 1...5 {
        print("Поток: \(i)")
        Thread.sleep(forTimeInterval: 1)
    }
}
thread.name = "MyBackgroundThread"
thread.start()
```

- Метод `start()` запускает поток.
    
- Код в замыкании выполняется **параллельно** основному потоку.
    

---

### 2.2 Использование selector

```swift
let thread = Thread(target: self, selector: #selector(runTask), object: nil)
thread.start()

@objc func runTask() {
    print("Поток выполняет задачу")
}
```

---

## 3. Свойства Thread

|Свойство|Описание|
|---|---|
|`isMainThread`|true, если поток главный (UI)|
|`isExecuting`|true, если поток выполняется|
|`isFinished`|true, если поток завершен|
|`name`|имя потока для отладки|
|`threadPriority`|приоритет потока (0.0 — 1.0)|

---

## 4. Пример использования

```swift
print("Главный поток: \(Thread.isMainThread)")

let backgroundThread = Thread {
    print("Работа в фоновом потоке: \(Thread.isMainThread)") // false
}
backgroundThread.start()
```

---

## 5. Потоки и RunLoop

- Каждый поток может иметь свой **RunLoop** для обработки событий и таймеров.
    
- На главном потоке RunLoop **встроен** и управляет UI.
    

Пример добавления таймера во фоновый поток:

```swift
let thread = Thread {
    let timer = Timer(timeInterval: 1.0, repeats: true) { _ in
        print("Таймер во фоновом потоке")
    }
    RunLoop.current.add(timer, forMode: .default)
    RunLoop.current.run()
}
thread.start()
```

---

## 6. Особенности

- Использование потоков вручную **редко** нужно в [[Swift]], чаще GCD и OperationQueue проще.
    
- Потоки требуют управления **синхронизацией** и внимательного подхода к данным.
    
- Фоновый поток не может изменять **UI** напрямую — только через главный поток.
    

---

## 7. Итог

- `Thread` = низкоуровневый инструмент для параллельного выполнения кода.
    
- На каждом потоке есть свой **RunLoop**.
    
- Можно запускать задачи через замыкания или selector.
    
- Для UI-потока RunLoop встроен, фоновый поток требует запуска RunLoop, если нужны таймеры или события.
    

---
