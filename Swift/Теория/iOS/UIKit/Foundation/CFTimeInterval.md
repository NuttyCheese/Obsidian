#core-animation #cftimeinterval #animation #timer #ios #swift #performance

---

## CFTimeInterval — Тип для представления времени

### Определение

**`CFTimeInterval`** — это тип данных в [[Core Foundation]] (`CF`), используемый для представления **интервалов времени** (в секундах) с плавающей точкой двойной точности. По сути, это `typealias` для `Double`, но с семантическим значением: переменная этого типа хранит временной интервал или абсолютное время (timestamp) .

В iOS-разработке `CFTimeInterval` активно используется в **[[Core Animation]]**, `CADisplayLink`, `CABasicAnimation`, `CACurrentMediaTime()`, а также в системных функциях, связанных со временем.

### Зачем это знать iOS-разработчику?

1.  **Анимации:** Управление длительностью (`duration`), задержкой (`beginTime`), временем начала анимации.
2.  **[[CADisplayLink]]:** Получение временных меток кадров для плавной анимации.
3.  **Таймеры:** [[CFRunLoopTimer]] использует `CFTimeInterval`.
4.  **Core Animation:** Практически все временные параметры (`duration`, `repeatDuration`, `timeOffset`).
5.  **Производительность:** Высокая точность (до наносекунд) критична для плавной анимации.

---

### Тип и его происхождение

```swift
public typealias CFTimeInterval = Double
```

- `CFTimeInterval` — это просто [[Double]].
- Используется для совместимости с Core Foundation и Core Animation.
- Значение измеряется в **секундах** (дробная часть — доли секунды).

---

### Основные константы

| Константа | Значение | Описание |
|---|---|---|
| **`0`** | `0.0` | Нулевая длительность (моментально) |
| **`greatestFiniteMagnitude`** | `DBL_MAX` | Бесконечность (для повторяющихся анимаций) |
| **`CACurrentMediaTime()`** | `CFTimeInterval` | Текущее время в секундах (монотонное, без учёта сна устройства) |

---

### Примеры использования

#### 1. **Длительность анимации [[UIView]]**

```swift
import UIKit

class AnimationDurationViewController: UIViewController {
    @IBOutlet weak var myView: UIView!
    
    @IBAction func startAnimation(_ sender: Any) {
        // Длительность анимации — 1.5 секунды
        let duration: CFTimeInterval = 1.5
        
        UIView.animate(withDuration: duration) {
            self.myView.center.x += 100
        }
    }
}
```

#### 2. **CABasicAnimation с CFTimeInterval**

```swift
class BasicAnimationViewController: UIViewController {
    @IBOutlet weak var myLayer: CALayer!
    
    func animateOpacity() {
        let animation = CABasicAnimation(keyPath: "opacity")
        animation.fromValue = 1.0
        animation.toValue = 0.0
        animation.duration = 2.0  // CFTimeInterval
        animation.beginTime = CACurrentMediaTime() + 1.0  // Старт через 1 секунду
        animation.fillMode = .forwards
        animation.isRemovedOnCompletion = false
        
        myLayer.add(animation, forKey: "fadeOut")
    }
}
```

#### 3. **[[CADisplayLink]] и CFTimeInterval**

```swift
class DisplayLinkViewController: UIViewController {
    var displayLink: CADisplayLink?
    var startTime: CFTimeInterval?
    var animationLayer: CALayer!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        animationLayer = CALayer()
        animationLayer.frame = CGRect(x: 50, y: 200, width: 100, height: 100)
        animationLayer.backgroundColor = UIColor.systemBlue.cgColor
        view.layer.addSublayer(animationLayer)
        
        displayLink = CADisplayLink(target: self, selector: #selector(updateAnimation))
        displayLink?.add(to: .main, forMode: .common)
        startTime = CACurrentMediaTime()
    }
    
    @objc func updateAnimation(displayLink: CADisplayLink) {
        guard let startTime = startTime else { return }
        let elapsed = displayLink.timestamp - startTime
        let duration: CFTimeInterval = 2.0
        
        if elapsed < duration {
            let progress = elapsed / duration
            let newX = 50 + (200 - 50) * CGFloat(progress)
            animationLayer.position.x = newX
        } else {
            displayLink.invalidate()
            self.displayLink = nil
        }
    }
    
    deinit {
        displayLink?.invalidate()
    }
}
```

---

### CACurrentMediaTime() vs Date().timeIntervalSince1970

| Характеристика | `CACurrentMediaTime()` | `Date().timeIntervalSince1970` |
|---|---|---|
| **Тип** | `CFTimeInterval` | `TimeInterval` (Double) |
| **Основа** | Монотонное время (от загрузки системы) | Wall-clock time (Unix epoch) |
| **Чувствительность к изменению системного времени** | Нет | Да |
| **Назначение** | Core Animation, CADisplayLink, измерение интервалов | Логирование, дата/время для пользователя |
| **Точность** | Высокая (наносекунды) | Высокая |

```swift
// Правильно для анимации
let start = CACurrentMediaTime()
animate()
let elapsed = CACurrentMediaTime() - start

// Правильно для отметки времени события
let timestamp = Date().timeIntervalSince1970
```

---

### Использование в таймерах (CFRunLoopTimer)

```swift
import Foundation

class TimerExample {
    func scheduleTimer() {
        let interval: CFTimeInterval = 2.0
        let fireTime = CFAbsoluteTimeGetCurrent() + interval
        
        let timer = CFRunLoopTimerCreateWithHandler(
            kCFAllocatorDefault,
            fireTime,
            interval,
            0,
            0
        ) { _ in
            print("Timer fired")
        }
        
        CFRunLoopAddTimer(CFRunLoopGetCurrent(), timer, .commonModes)
    }
}
```

---

### Преобразования

```swift
let duration: CFTimeInterval = 1.5

// В миллисекунды (для логирования)
let milliseconds = duration * 1000  // 1500.0

// В TimeInterval (для Date)
let timeInterval = TimeInterval(duration)

// В Int (целые секунды)
let seconds = Int(duration)  // 1
```

---

### CFTimeInterval в анимациях с повторением

```swift
class RepeatingAnimationViewController: UIViewController {
    @IBOutlet weak var myView: UIView!
    
    func startPulseAnimation() {
        let animation = CABasicAnimation(keyPath: "transform.scale")
        animation.fromValue = 1.0
        animation.toValue = 1.2
        animation.duration = 0.5
        animation.autoreverses = true
        animation.repeatCount = .infinity
        animation.repeatDuration = 5.0  // Останавливается через 5 секунд
        myView.layer.add(animation, forKey: "pulse")
    }
}
```

---

### Работа с Core Animation и временем

```swift
class ComplexAnimationViewController: UIViewController {
    @IBOutlet weak var myLayer: CALayer!
    
    func setupAnimation() {
        let animation = CAKeyframeAnimation(keyPath: "position.x")
        animation.values = [50, 150, 250, 150, 50]
        animation.keyTimes = [0, 0.25, 0.5, 0.75, 1]
        animation.duration = 4.0
        animation.timeOffset = 1.0  // Старт со смещением
        animation.speed = 2.0       // Ускоренная анимация
        
        myLayer.add(animation, forKey: "move")
    }
}
```

---

### Измерение производительности

```swift
class PerformanceMeasureViewController: UIViewController {
    func measurePerformance() {
        let start = CACurrentMediaTime()
        
        // Измеряемый код
        for _ in 0..<10000 {
            // какая-то работа
        }
        
        let end = CACurrentMediaTime()
        let elapsed = end - start
        print("Operation took \(elapsed * 1000) ms")
    }
}
```

---

### Best Practices

1.  **Используйте `CACurrentMediaTime()` для измерения интервалов** — она не зависит от изменения системного времени.
2.  **Не храните `CFTimeInterval` как `Int`** — потеряете дробную часть.
3.  **Для анимаций с бесконечным повторением используйте `repeatCount = .greatestFiniteMagnitude`** вместо `infinity`.
4.  **Проверяйте `elapsed` на отрицательные значения** — при изменении системного времени между вызовами.
5.  **Используйте `CFAbsoluteTimeGetCurrent()` для абсолютных отметок** (CFRunLoopTimer).

---

### Итог

**`CFTimeInterval`** — это специализированный тип для работы со временем в Core Animation и Core Foundation. Он обеспечивает:

1.  **Высокую точность** (доли секунды) для плавной анимации.
2.  **Монотонность** (через `CACurrentMediaTime()`) для измерения интервалов.
3.  **Единообразие API** между Core Animation, CADisplayLink и таймерами.
4.  **Удобство** при работе с длительностью анимаций и задержками.

Понимание `CFTimeInterval` необходимо для создания точных, высокопроизводительных анимаций и корректного измерения времени в [[iOS]]-приложениях .