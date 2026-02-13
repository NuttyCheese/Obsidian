В Swift, `Selector` - это тип данных, который представляет собой ссылку на метод или функцию объекта. Он используется в Objective-C и Cocoa для обозначения методов объектов, которые можно вызвать динамически во время выполнения программы. Хотя в Swift напрямую вызывать методы через `Selector` можно, но использование `Selector` не так распространено, как в [[Objective-C]].

`Selector` часто используется в контексте целей (target) и селекторов (selector) при настройке целей для обработки событий, таких как нажатие кнопки, или при добавлении методов к цели для выполнения определенных действий.

Пример использования `Selector` в [[Swift]]:
```swift
import UIKit

class MyViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Создание кнопки
        let button = UIButton(frame: CGRect(x: 100, y: 100, width: 200, height: 50))
        button.setTitle("Нажми меня", for: .normal)
        button.setTitleColor(.blue, for: .normal)
        view.addSubview(button)
        
        // Назначение метода для обработки нажатия кнопки с использованием Selector
        button.addTarget(self, action: #selector(buttonPressed(_:)), for: .touchUpInside)
    }
    
    // Метод, который будет вызван при нажатии кнопки
    @objc func buttonPressed(_ sender: UIButton) {
        print("Кнопка нажата")
    }
}

```
В этом примере метод `buttonPressed(_:)` назначается как обработчик события нажатия кнопки через `Selector`. Метод должен быть помечен атрибутом `@objc`, чтобы быть доступным для вызова через `Selector`. В момент нажатия кнопки вызывается метод, указанный в `Selector`, и выполняется соответствующее действие.