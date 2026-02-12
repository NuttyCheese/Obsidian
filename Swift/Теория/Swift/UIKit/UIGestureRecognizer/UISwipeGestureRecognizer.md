`UISwipeGestureRecognizer` используется для обнаружения быстрого свайпа в указанном направлении на представлении. Этот жест часто используется для реализации функциональности перемещения между экранами или для выполнения определенных действий при свайпе пользователя.

Пример использования `UISwipeGestureRecognizer` в [[iOS]]-приложении:

```swift
import UIKit

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Создаем UISwipeGestureRecognizer для свайпа вправо
        let swipeRightGesture = UISwipeGestureRecognizer(target: self, action: #selector(handleSwipe(_:)))
        swipeRightGesture.direction = .right
        
        // Добавляем UISwipeGestureRecognizer к представлению
        view.addGestureRecognizer(swipeRightGesture)
        
        // Создаем UISwipeGestureRecognizer для свайпа влево
        let swipeLeftGesture = UISwipeGestureRecognizer(target: self, action: #selector(handleSwipe(_:)))
        swipeLeftGesture.direction = .left
        
        // Добавляем UISwipeGestureRecognizer к представлению
        view.addGestureRecognizer(swipeLeftGesture)
    }
    
    @objc func handleSwipe(_ sender: UISwipeGestureRecognizer) {
        // Обработка свайпа
        if sender.direction == .right {
            print("Swipe right detected")
        } else if sender.direction == .left {
            print("Swipe left detected")
        }
    }
}
```

В этом примере создаются два `UISwipeGestureRecognizer`: один для свайпа вправо (`.right`), а второй для свайпа влево (`.left`). Затем каждый из них добавляется к представлению контроллера. При обнаружении свайпа вызывается метод `handleSwipe(_:)`, который определяет направление свайпа и выводит соответствующее сообщение.

`UISwipeGestureRecognizer` также поддерживает настройки, такие как количество пальцев (`numberOfTouchesRequired`) и скорость свайпа (`velocity`). Эти свойства могут быть настроены в соответствии с требованиями вашего приложения.