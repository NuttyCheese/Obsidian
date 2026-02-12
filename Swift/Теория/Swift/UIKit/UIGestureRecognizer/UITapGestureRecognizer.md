`UITapGestureRecognizer` используется для обнаружения одиночного касания (тапа) на представлении. Это один из наиболее распространенных жестов в iOS-приложениях и часто используется для активации определенного действия при нажатии на элемент интерфейса.

Пример использования `UITapGestureRecognizer` в [[iOS]]-приложении:

```swift
import UIKit

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Создаем UITapGestureRecognizer
        let tapGesture = UITapGestureRecognizer(target: self, action: #selector(handleTap(_:)))
        
        // Настраиваем количество нажатий
        tapGesture.numberOfTapsRequired = 1
        
        // Настраиваем количество пальцев
        tapGesture.numberOfTouchesRequired = 1
        
        // Добавляем UITapGestureRecognizer к представлению
        view.addGestureRecognizer(tapGesture)
    }
    
    @objc func handleTap(_ sender: UITapGestureRecognizer) {
        // Обработка тапа
        print("Tap detected")
    }
}
```

В этом примере создается `UITapGestureRecognizer` и добавляется к представлению контроллера. При каждом обнаружении тапа вызывается метод `handleTap(_:`, который выводит сообщение о том, что тап был обнаружен.

`UITapGestureRecognizer` также поддерживает дополнительные настройки, такие как количество нажатий (`numberOfTapsRequired`) и количество пальцев (`numberOfTouchesRequired`). Эти свойства можно настроить в соответствии с требованиями вашего приложения.