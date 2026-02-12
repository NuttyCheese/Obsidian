# Responder Chain в UIKit

**Responder Chain** — это механизм, с помощью которого [[iOS]] доставляет события (touch, gesture, motion, action) от системы к нужному объекту в иерархии приложения.

Это **основа**:

- обработки нажатий
    
- кнопок
    
- жестов
    
- `target–action`
    
- меню, `UICommand`, клавиатуры
    
- и даже `sendAction(_:)`
    

Если ты понимаешь responder chain — ты понимаешь UIKit.

---

# 1. Кто такой Responder

Все объекты, участвующие в обработке событий, наследуются от:

```swift
UIResponder
```

К ним относятся:

- `UIView`
    
- `UIViewController`
    
- `UIWindow`
    
- `UIApplication`
    

Каждый [[UIResponder]] знает, кто следующий в цепочке:

```swift
var next: UIResponder? { get }
```

Это и есть **цепочка**.

---

# 2. Базовая цепочка

Для обычного экрана цепочка выглядит так:

```
UIView (кнопка)
  ↓
Superview
  ↓
...
  ↓
UIViewController
  ↓
UIWindow
  ↓
UIApplication
```

Если объект не обработал событие — оно передаётся дальше по `next`.

---

# 3. Откуда вообще берётся событие

Жизненный путь нажатия:

```
Палец
 ↓
iOS Kernel
 ↓
SpringBoard
 ↓
UIApplication
 ↓
UIWindow
 ↓
hitTest(_:with:)
 ↓
Найденная UIView
 ↓
Responder Chain
```

UIKit сначала определяет **какой View был нажат** через `hitTest`, а потом уже запускает responder chain.

---

# 4. [[hitTest]] и выбор первой view

UIKit ищет самую глубокую view под пальцем:

```swift
window.hitTest(point, with: event)
```

Упрощённо:

```swift
func hitTest(_ point: CGPoint) -> UIView? {
    for subview in subviews.reversed() {
        if subview.frame.contains(point) {
            return subview.hitTest(convertedPoint)
        }
    }
    return self
}
```

Это **не responder chain** — это поиск стартовой точки.

Responder chain начинается **после**.

---

# 5. Простейший пример

```swift
class MyView: UIView {
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        print("MyView")
        super.touchesBegan(touches, with: event)
    }
}

class MyViewController: UIViewController {
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        print("ViewController")
        super.touchesBegan(touches, with: event)
    }
}
```

При тапе по `MyView`:

```
MyView
ViewController
UIWindow
UIApplication
```

Если ты **не вызовешь** `super`, цепочка **оборвётся**.

---

# 6. Как кнопки работают через Responder Chain

[[UIButton]] не вызывает метод напрямую. Он делает:

```swift
UIApplication.shared.sendAction(
    #selector(buttonTapped),
    to: target,
    from: self,
    for: event
)
```

Если `target == nil`, UIKit ищет обработчик по responder chain.

Это значит:

```swift
@objc func buttonTapped()
```

может быть:

- в [[UIView]]
    
- в [[UIViewController]]
    
- даже в `AppDelegate`
    

---

# 7. Магия target = nil

```swift
button.addTarget(nil, action: #selector(save), for: .touchUpInside)
```

UIKit делает:

```
button
↓
superview
↓
view controller
↓
window
↓
app
```

И вызывает первый объект, где есть `save`.

Это позволяет делать **децентрализованную архитектуру UI**.

---

# 8. Как ViewController попадает в цепочку

По умолчанию:

```
UIView → UIViewController → UIWindow
```

UIKit автоматически связывает:

```swift
view.next == viewController
viewController.next == window
```

Поэтому ViewController может ловить:

- кнопки
    
- жесты
    
- клавиатуру
    
- меню
    

без прямых ссылок.

---

# 9. Жизненный цикл события

Для touch:

```
touchesBegan
→ touchesMoved
→ touchesEnded / touchesCancelled
```

Каждый этап идёт по responder chain.

Если объект «съел» событие и не вызвал `super` — дальше оно не пойдёт.

---

# 10. Gesture Recognizers и Responder Chain

Жесты сидят **между hitTest и responder chain**.

Пайплайн:

```
hitTest → UIView
    ↓
Gesture Recognizers
    ↓ (если не распознали)
touchesBegan → responder chain
```

Поэтому:

- жест может «перехватить» событие
    
- touches не придут вообще
    

---

# 11. Почему это фундамент UIKit

Responder Chain позволяет:

- кнопкам не знать про контроллеры
    
- view не иметь ссылок на владельцев
    
- меню, клавиатуре, командам работать глобально
    

Это **архитектурная развязка**, заложенная ещё в NeXTSTEP — и она переживёт любые UI-фреймворки поверх UIKit.

---

# Ментальная модель

Responder Chain — это не «обработка тапа».  
Это **маршрут, по которому UIKit доставляет намерение пользователя** от пикселя до логики.

Сначала находится место,  
потом ищется смысл,  
и только потом вызывается код.
