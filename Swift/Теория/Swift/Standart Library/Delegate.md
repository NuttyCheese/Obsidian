#standart_library #Swift 
## 📘 Определение

**Delegate** — паттерн проектирования в [[Swift]] (и [[UIKit]]), позволяющий одному объекту **передавать ответственность за выполнение определённых действий другому объекту**.  
Используется для обратной связи между объектами без жёсткой связи между ними. Часто применяется в **UIKit** ([[Swift/Теория/Swift/UIKit/TableView/UITableView]], [[UITextField]] и др.).

---

## 🔹 Примеры кода

### 1. Простая реализация делегата

```swift
protocol PrinterDelegate: AnyObject {
    func didPrint(message: String)
}

class Printer {
    weak var delegate: PrinterDelegate?

    func printMessage() {
        delegate?.didPrint(message: "Hello, Delegate!")
    }
}

class Receiver: PrinterDelegate {
    func didPrint(message: String) {
        print("Получено сообщение:", message)
    }
}

let printer = Printer()
let receiver = Receiver()
printer.delegate = receiver
printer.printMessage()
```

---

### 2. Делегат с UIKit (UITextField)

```swift
import UIKit

class ViewController: UIViewController, UITextFieldDelegate {
    let textField = UITextField()

    override func viewDidLoad() {
        super.viewDidLoad()
        textField.delegate = self
    }

    func textFieldShouldReturn(_ textField: UITextField) -> Bool {
        print("Нажата клавиша Return:", textField.text ?? "")
        return true
    }
}
```

---

### 3. Делегат с кастомным событием

```swift
protocol DownloadDelegate: AnyObject {
    func downloadDidFinish()
}

class Downloader {
    weak var delegate: DownloadDelegate?

    func startDownload() {
        print("Скачивание...")
        delegate?.downloadDidFinish()
    }
}

class Controller: DownloadDelegate {
    func downloadDidFinish() {
        print("Скачивание завершено")
    }
}

let downloader = Downloader()
let controller = Controller()
downloader.delegate = controller
downloader.startDownload()
```

---

### 4. Делегат с optional методами через `@objc` (UIKit-стиль)

```swift
@objc protocol CustomDelegate {
    @objc optional func didTapButton()
}

class ButtonHandler {
    weak var delegate: CustomDelegate?
    func tap() {
        delegate?.didTapButton?()
    }
}

class ViewController: CustomDelegate {
    func setup() {
        let handler = ButtonHandler()
        handler.delegate = self
        handler.tap()
    }
}
```

---

### 5. Делегат с передачей данных между экранами

```swift
protocol ColorSelectionDelegate: AnyObject {
    func didSelectColor(_ color: UIColor)
}

class ColorPickerViewController: UIViewController {
    weak var delegate: ColorSelectionDelegate?

    func select(color: UIColor) {
        delegate?.didSelectColor(color)
        dismiss(animated: true)
    }
}

class MainViewController: UIViewController, ColorSelectionDelegate {
    func didSelectColor(_ color: UIColor) {
        view.backgroundColor = color
    }

    func openPicker() {
        let picker = ColorPickerViewController()
        picker.delegate = self
        present(picker, animated: true)
    }
}
```
