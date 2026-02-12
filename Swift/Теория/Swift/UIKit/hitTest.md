Метод `hitTest` - это метод, который используется в [[UIKit]] для определения представления, которое находится под определенной точкой в координатах экрана. Этот метод позволяет рекурсивно пройти по иерархии представлений и найти представление, которое находится под указанной точкой, а также определить, на какой глубине оно находится.

Связь между методом `hitTest` и UIKit заключается в том, что UIKit использует этот метод для обработки касаний и других событий пользовательского ввода. Когда пользователь касается экрана или выполняет другие действия в приложении, UIKit вызывает метод `hitTest` на корневом представлении иерархии для определения, какое представление находится под указанной точкой.

Пример использования метода `hitTest` в UIKit:

```swift
import UIKit

class MyView: UIView {
    override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
        // Проверяем, находится ли точка касания внутри границ представления MyView
        if self.bounds.contains(point) {
            // Возвращаем представление MyView, так как оно находится под точкой касания
            return self
        } else {
            // Если точка касания находится за пределами границ представления MyView,
            // передаем запрос на определение представления по цепочке родительских представлений
            return super.hitTest(point, with: event)
        }
    }
}

// Создание экземпляра представления
let myView = MyView(frame: CGRect(x: 0, y: 0, width: 200, height: 200))
myView.backgroundColor = UIColor.blue

// Добавление представления на экран
let viewController = UIViewController()
viewController.view.addSubview(myView)

// Точка касания
let touchPoint = CGPoint(x: 100, y: 100)

// Поиск представления, находящегося под точкой касания
if let viewBelowTouch = myView.hitTest(touchPoint, with: nil) {
    print("Представление \(viewBelowTouch) находится под точкой касания.")
} else {
    print("Представление не найдено под точкой касания.")
}
```

В этом примере создается пользовательское представление `MyView`, которое переопределяет метод `hitTest` для определения, находится ли точка касания внутри его границ. Если точка касания находится в пределах границ представления, метод возвращает это представление. Если точка касания находится за пределами представления, запрос на определение представления передается родительскому представлению.

Ниже — цельная глава про **Hit-Testing в UIKit**. Это та часть UIKit, которая решает _«куда именно пользователь ткнул»_ ещё до того, как начнётся responder chain.

---

UIKit использует его, чтобы определить:

> _какой объект должен первым получить событие._

Responder chain начинается **после** hit-test.

---

# Общий жизненный цикл тапа

Когда пользователь касается экрана:

```
Палец
 ↓
iOS
 ↓
UIApplication
 ↓
UIWindow
 ↓
hitTest(_:with:)
 ↓
Конкретная UIView
 ↓
Gesture recognizers
 ↓
Responder Chain
```

Hit-test — это геометрический поиск точки в иерархии view.

---

# 1. С чего всё начинается — [[UIWindow]]

Любой touch приходит в `UIWindow`.

Он вызывает:

```swift
let view = window.hitTest(point, with: event)
```

UIKit спрашивает у окна:

> “Какая view находится под этой точкой?”

---

# 2. Как работает hitTest

Упрощённая логика:

```swift
func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
    if !isUserInteractionEnabled || isHidden || alpha < 0.01 {
        return nil
    }

    if !self.point(inside: point, with: event) {
        return nil
    }

    for subview in subviews.reversed() {
        let converted = convert(point, to: subview)
        if let hit = subview.hitTest(converted, with: event) {
            return hit
        }
    }

    return self
}
```

UIKit идёт:

1. Сначала проверяет саму view
    
2. Потом её сабвью (сверху вниз)
    
3. Возвращает **самую глубокую подходящую**
    

---

# 3. Простейший пример

```swift
let red = UIView(frame: CGRect(x: 0, y: 0, width: 200, height: 200))
let blue = UIView(frame: CGRect(x: 50, y: 50, width: 100, height: 100))

red.addSubview(blue)
```

Если ты нажмёшь в центр:

```
hitTest → blue
```

Если в угол:

```
hitTest → red
```

UIKit всегда выбирает **самую вложенную** view под пальцем.

---

# 4. Почему некоторые view не получают нажатия

View автоматически исключается из hit-test, если:

```swift
isHidden == true
alpha < 0.01
isUserInteractionEnabled == false
```

Например:

```swift
button.isUserInteractionEnabled = false
```

→ она станет **прозрачной для касаний**, как будто её нет.

---

# 5. point(inside:)

Перед тем как смотреть subviews, UIKit вызывает:

```swift
func point(inside point: CGPoint, with event: UIEvent?) -> Bool
```

По умолчанию:

```swift
return bounds.contains(point)
```

Ты можешь расширять зоны нажатия:

```swift
override func point(inside point: CGPoint, with event: UIEvent?) -> Bool {
    let bigger = bounds.insetBy(dx: -20, dy: -20)
    return bigger.contains(point)
}
```

Так делают кнопки с маленькой иконкой.

---

# 6. Кастомный hitTest

Можно полностью изменить логику:

```swift
override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
    let hit = super.hitTest(point, with: event)

    if hit == self {
        return nil // пропускаем сквозь себя
    }

    return hit
}
```

Используется для:

- клика сквозь overlay
    
- кастомных контейнеров
    
- floating UI
    

---

# 7. Пример: кнопка поверх карты

```swift
override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
    if button.frame.contains(point) {
        return button
    }
    return mapView
}
```

Тапы по кнопке идут в кнопку, всё остальное — в карту.

---

# 8. Как hit-test связан с gesture recognizers

После того как view найдена:

```
hitTest → UIView
     ↓
Gesture recognizers этой view и её родителей
     ↓
touchesBegan → responder chain
```

Поэтому:

- если жест распознал событие — `touches` не придут
    
- если нет — оно пойдёт дальше
    

---

# 9. Важный архитектурный смысл

Hit-test решает:

> **где произошло событие**

Responder Chain решает:

> **кто его обработает**

Они разделены специально.  
Это позволяет UIKit быть:

- точным (геометрия)
    
- гибким (архитектура)
    

И именно поэтому UIKit живёт десятилетиями и всё ещё масштабируется — от кнопки до сложных интерактивных экранов.
