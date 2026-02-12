#tests #Swift 
## 📘 Определение

**iOSSnapshotTestCase** — фреймворк для **снапшот-тестирования UI в [[iOS]]-приложениях**.  
Позволяет **сравнивать визуальное представление [[Swift/Теория/Swift/UIKit/UIView]] или ViewController** с эталонным изображением, чтобы обнаруживать визуальные регрессии после изменений кода.  
Относится к **тестированию UI** и используется совместно с **[[XCTest]]**.

---

## 🔹 Примеры кода

### 1. Подключение фреймворка

```ruby
# Podfile
target 'MyAppTests' do
  pod 'iOSSnapshotTestCase'
end
```

---

### 2. Простейший тест ViewController

```swift
import FBSnapshotTestCase
@testable import MyApp

class MyViewControllerTests: FBSnapshotTestCase {
    
    override func setUp() {
        super.setUp()
        self.recordMode = false // true для записи эталонного снимка
    }
    
    func testMainViewControllerSnapshot() {
        let vc = MainViewController()
        vc.loadViewIfNeeded()
        
        FBSnapshotVerifyView(vc.view)
    }
}
```

---

### 3. Тест UIView

```swift
func testCustomViewSnapshot() {
    let view = CustomView(frame: CGRect(x: 0, y: 0, width: 200, height: 100))
    view.backgroundColor = .white
    
    FBSnapshotVerifyView(view)
}
```

---

### 4. Тест с разными устройствами/разрешениями

```swift
func testView_iPhoneSE() {
    let view = CustomView(frame: CGRect(x: 0, y: 0, width: 320, height: 568))
    view.backgroundColor = .white
    FBSnapshotVerifyView(view)
}
```

---

### 5. Тест с задержкой (для анимаций)

```swift
func testViewAfterAnimation() {
    let view = CustomView(frame: CGRect(x: 0, y: 0, width: 200, height: 100))
    view.startAnimation()
    
    // Даем анимации завершиться
    RunLoop.current.run(until: Date().addingTimeInterval(1))
    
    FBSnapshotVerifyView(view)
}
```
