`UIPinchGestureRecognizer` используется для обнаружения жеста масштабирования (прижатия или разжатия двумя пальцами) на представлении. Этот жест часто используется для реализации функциональности изменения масштаба объектов в пользовательском интерфейсе, например, для масштабирования изображений или элементов управления.

Пример использования `UIPinchGestureRecognizer` в [[iOS]]-приложении:

```swift
import UIKit

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Создаем UIPinchGestureRecognizer
        let pinchGesture = UIPinchGestureRecognizer(target: self, action: #selector(handlePinch(_:)))
        
        // Добавляем UIPinchGestureRecognizer к представлению
        view.addGestureRecognizer(pinchGesture)
    }
    
    @objc func handlePinch(_ sender: UIPinchGestureRecognizer) {
        // Обработка масштабирования
        let scale = sender.scale // Масштаб
        
        // Применяем масштабирование к представлению
        if let scaledView = sender.view {
            scaledView.transform = scaledView.transform.scaledBy(x: scale, y: scale)
        }
        
        // Сбрасываем начальные параметры после обработки жеста
        sender.scale = 1
    }
}
```

В этом примере создается `UIPinchGestureRecognizer` и добавляется к представлению контроллера. При каждом обнаружении жеста масштабирования вызывается метод `handlePinch(_:)`, который применяет масштабирование к представлению на основе значения масштаба, определенного `UIPinchGestureRecognizer`. После обработки жеста начальный масштаб сбрасывается до единицы.

`UIPinchGestureRecognizer` также поддерживает дополнительные настройки, такие как количество пальцев (`numberOfTouchesRequired`) и минимальное и максимальное количество пальцев (`minimumNumberOfTouches` и `maximumNumberOfTouches`). Эти свойства могут быть настроены в соответствии с требованиями вашего приложения.