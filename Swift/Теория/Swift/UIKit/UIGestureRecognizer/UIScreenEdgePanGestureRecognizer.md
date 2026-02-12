`UIScreenEdgePanGestureRecognizer` используется для обнаружения свайпа с края экрана на представлении. Этот жест часто используется для создания пользовательского интерфейса с боковым меню или переходом между экранами по свайпу с края.

Пример использования `UIScreenEdgePanGestureRecognizer` в [[iOS]]-приложении:

```swift
import UIKit

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Создаем UIScreenEdgePanGestureRecognizer для свайпа с правого края экрана
        let swipeRightGesture = UIScreenEdgePanGestureRecognizer(target: self, action: #selector(handleSwipe(_:)))
        swipeRightGesture.edges = .right
        
        // Добавляем UIScreenEdgePanGestureRecognizer к представлению
        view.addGestureRecognizer(swipeRightGesture)
        
        // Создаем UIScreenEdgePanGestureRecognizer для свайпа с левого края экрана
        let swipeLeftGesture = UIScreenEdgePanGestureRecognizer(target: self, action: #selector(handleSwipe(_:)))
        swipeLeftGesture.edges = .left
        
        // Добавляем UIScreenEdgePanGestureRecognizer к представлению
        view.addGestureRecognizer(swipeLeftGesture)
    }
    
    @objc func handleSwipe(_ sender: UIScreenEdgePanGestureRecognizer) {
        // Обработка свайпа с края экрана
        if sender.edges == .right {
            print("Swipe from right edge detected")
        } else if sender.edges == .left {
            print("Swipe from left edge detected")
        }
    }
}
```

В этом примере создаются два `UIScreenEdgePanGestureRecognizer`: один для свайпа с правого края экрана (`.right`), а второй для свайпа с левого края экрана (`.left`). Затем каждый из них добавляется к представлению контроллера. При обнаружении свайпа с края экрана вызывается метод `handleSwipe(_:)`, который определяет, с какого края был совершен свайп, и выводит соответствующее сообщение.

`UIScreenEdgePanGestureRecognizer` также поддерживает настройки, такие как ширина области края экрана (`UIScreenEdgePanGestureRecognizer.edges`) и скорость свайпа (`velocity`). Эти свойства могут быть настроены в соответствии с требованиями вашего приложения.