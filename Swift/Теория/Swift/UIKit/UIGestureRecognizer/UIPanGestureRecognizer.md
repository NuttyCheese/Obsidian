`UIPanGestureRecognizer` используется для обнаружения жеста перемещения пальца по экрану (перетаскивания) на представлении. Этот жест часто используется для реализации функциональности перемещения объектов или для обработки пользовательского ввода, требующего перемещения пальца по экрану.

Пример использования `UIPanGestureRecognizer` в [[iOS]]-приложении:

```swift
import UIKit

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Создаем UIPanGestureRecognizer
        let panGesture = UIPanGestureRecognizer(target: self, action: #selector(handlePan(_:)))
        
        // Добавляем UIPanGestureRecognizer к представлению
        view.addGestureRecognizer(panGesture)
    }
    
    @objc func handlePan(_ sender: UIPanGestureRecognizer) {
        // Получаем текущее положение пальца на экране
        let translation = sender.translation(in: view)
        
        // Обрабатываем перемещение пальца
        if let draggedView = sender.view {
            // Изменяем положение представления в соответствии с перемещением пальца
            draggedView.center = CGPoint(x: draggedView.center.x + translation.x, y: draggedView.center.y + translation.y)
        }
        
        // Сбрасываем значение перемещения для следующего цикла жеста
        sender.setTranslation(CGPoint.zero, in: view)
    }
}
```

В этом примере создается `UIPanGestureRecognizer` и добавляется к представлению контроллера. При каждом обнаружении жеста перемещения пальца по экрану вызывается метод `handlePan(_:)`, который изменяет положение представления в соответствии с перемещением пальца. После обработки жеста значение перемещения сбрасывается для следующего цикла жеста.

`UIPanGestureRecognizer` также поддерживает дополнительные настройки, такие как количество пальцев (`minimumNumberOfTouches`, `maximumNumberOfTouches`) и направление движения (`direction`). Эти свойства могут быть настроены в соответствии с требованиями вашего приложения.