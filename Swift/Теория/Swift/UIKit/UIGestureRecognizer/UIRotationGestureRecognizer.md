`UIRotationGestureRecognizer` используется для обнаружения жеста вращения двумя пальцами на представлении. Этот жест часто используется для реализации функциональности изменения размера или вращения объектов в пользовательском интерфейсе.

Пример использования `UIRotationGestureRecognizer` в [[iOS]]-приложении:

```swift
import UIKit

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Создаем UIRotationGestureRecognizer
        let rotationGesture = UIRotationGestureRecognizer(target: self, action: #selector(handleRotation(_:)))
        
        // Добавляем UIRotationGestureRecognizer к представлению
        view.addGestureRecognizer(rotationGesture)
    }
    
    @objc func handleRotation(_ sender: UIRotationGestureRecognizer) {
        // Обработка вращения
        let rotationAngle = sender.rotation // Угол вращения в радианах
        
        // Применяем вращение к представлению
        if let rotatedView = sender.view {
            rotatedView.transform = rotatedView.transform.rotated(by: rotationAngle)
        }
        
        // Сбрасываем начальный угол после обработки жеста
        sender.rotation = 0
    }
}
```

В этом примере создается `UIRotationGestureRecognizer` и добавляется к представлению контроллера. При каждом обнаружении жеста вращения вызывается метод `handleRotation(_:)`, который применяет вращение к представлению на основе угла вращения, определенного `UIRotationGestureRecognizer`. После обработки жеста начальный угол сбрасывается до нуля.

`UIRotationGestureRecognizer` также поддерживает дополнительные настройки, такие как количество пальцев (`numberOfTouchesRequired`) и минимальное и максимальное количество пальцев (`minimumNumberOfTouches` и `maximumNumberOfTouches`). Эти свойства могут быть настроены в соответствии с требованиями вашего приложения.