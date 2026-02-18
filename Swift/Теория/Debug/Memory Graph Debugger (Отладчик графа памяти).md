![Image](https://i.sstatic.net/ZcS7b.png)

![Image](https://i.sstatic.net/fiVPB.png)

![Image](https://agostini.tech/uploads/2018/11/Screenshot-2018-11-17-at-16.49.06-1024x706.png)

![Image](https://i.sstatic.net/l1J3g.jpg)

## Что такое Memory Graph Debugger

**Memory Graph Debugger** — это инструмент в [[Xcode]], который позволяет:

- увидеть все объекты в памяти
    
- посмотреть, кто кого удерживает
    
- найти [[retain cycle]]
    
- обнаружить утечки памяти
    
- понять, почему объект не освобождается
    

Проще говоря:

> Это визуальная карта всей памяти вашего приложения в момент выполнения.

Работает во время Debug-сессии.

---

# 📍 Где находится

1. Запустить приложение в Debug
    
2. В Debug Area нажать кнопку 🧠 (Debug Memory Graph)
    
3. Xcode остановит выполнение и построит граф объектов
    

Горячая кнопка появляется в нижней панели.

---

# 🧱 Что показывает Memory Graph

---

## 1️⃣ Объекты в памяти

- ViewController
    
- View
    
- Model
    
- [[Closure]]
    
- [[Delegate]]
    
- [[Singleton]]
    

Каждый объект — это узел графа.

---

## 2️⃣ Связи между объектами

Стрелки показывают:

- [[strong]] reference
    
- [[weak]] reference
    
- [[unowned]]
    

Это ключ к поиску [[retain cycle]].

---

## 3️⃣ Утечки (Leaks)

Xcode автоматически подсвечивает:

- объекты, которые не должны существовать
    
- потенциальные retain cycle
    

Обычно отмечаются фиолетовой иконкой ⚠️

---

# 🔬 Как работает под капотом

Когда ты нажимаешь кнопку:

1. Приложение ставится на паузу
    
2. Xcode анализирует [[heap]]
    
3. Строится граф ссылок
    
4. Выявляются сильные циклические связи
    

Он не просто показывает объекты — он анализирует ownership.

---

# 🧪 Практический сценарий №1

## ViewController не освобождается

### Симптом:

- экран закрыт
    
- deinit не вызывается
    
- память растёт
    

### Алгоритм:

1. Открыть экран
    
2. Закрыть его
    
3. Нажать Memory Graph
    
4. Найти ViewController в поиске
    
5. Посмотреть, кто его удерживает
    

Частые причины:

- strong [[delegate]]
    
- strong closure
    
- [[NotificationCenter]]
    
- [[Timer]]
    
- [[Combine]] subscription
    

---

# 🧪 Практический сценарий №2

## [[Retain cycle]] через closure

Пример проблемы:

```swift
class MyVC: UIViewController {

    var completion: (() -> Void)?

    func setup() {
        completion = {
            self.view.backgroundColor = .red
        }
    }
}
```

Memory Graph покажет:

```
MyVC
 └── completion
      └── closure
           └── strong → MyVC
```

Решение:

```swift
completion = { [weak self] in
    self?.view.backgroundColor = .red
}
```

---

# 🧪 Практический сценарий №3

## Delegate удерживает объект

Если забыть:

```swift
protocol SomeDelegate: AnyObject
```

и написать:

```swift
var delegate: SomeDelegate?
```

будет strong reference.

Memory Graph покажет цикл:

```
Parent → Child
Child → Parent
```

---

# 🧭 Навигация по Memory Graph

---

## 🔎 Поиск объекта

В строке поиска можно ввести:

```
LoginViewController
NetworkService
MyCustomCell
```

---

## 📍 Выбор объекта

При выборе:

- слева — граф
    
- справа — детали объекта
    
- ниже — список ссылок (who retains it)
    

---

## 🔁 Показать Retain Cycle

Если есть цикл — Xcode покажет цепочку.

Это главный инструмент для [[iOS]]-разработчика.

---

# 🚀 Продвинутые случаи

---

## 🔥 1. NotificationCenter

Если забыть удалить observer:

```swift
NotificationCenter.default.addObserver(...)
```

Memory Graph покажет удержание через internal observer storage.

---

## 🔥 2. Timer

```swift
Timer.scheduledTimer(...)
```

Timer удерживает target сильно.

---

## 🔥 3. Combine

Если не отменить:

```swift
cancellable = publisher.sink { ... }
```

Graph покажет цепочку через AnyCancellable.

---

## 🔥 4. [[SwiftUI]] + [[UIKit]]

HostingController может удерживаться через environment.

Graph помогает это увидеть.

---

# 📊 Когда использовать Memory Graph

| Сценарий                 | Использовать   |
| ------------------------ | -------------- |
| [[deinit]] не вызывается | Да             |
| Память растёт            | Да             |
| Экран не освобождается   | Да             |
| Краш по памяти           | Да             |
| Просто проверить код     | Не обязательно |

---

# ⚠️ Ограничения

- Работает только в Debug
    
- Не всегда ловит все утечки
    
- Не заменяет Instruments (Leaks tool)
    

Для глубокого анализа — Instruments.

---

# 🧠 Правильный алгоритм поиска утечки

1. Воспроизвести проблему
    
2. Закрыть экран
    
3. Нажать Memory Graph
    
4. Найти объект
    
5. Проверить retain path
    
6. Найти сильную ссылку
    
7. Исправить код
    
8. Повторить проверку
    

---

# 🎯 Почему это критично

- UIKit активно работает с [[reference type]]
    
- Большинство проблем — это ownership
    
- Retain cycle — одна из самых частых причин багов
    
- Понимание памяти = зрелость разработчика
    

---
