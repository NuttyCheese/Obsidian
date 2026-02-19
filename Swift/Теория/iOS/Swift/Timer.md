**`Timer`** (ранее `NSTimer`) — это класс из **Foundation**, который позволяет запускать код **через заданный интервал времени** (однократно или повторно). Он интегрируется с **RunLoop** и работает в том потоке, в котором был создан (обычно главный поток для UI-задач).

### Ключевые особенности Timer в 2026 году

- **Блок-версия** (`scheduledTimer(withTimeInterval:repeats:)`): самый рекомендуемый и безопасный способ с iOS 10+
- **Selector-версия** (`scheduledTimer(timeInterval:target:selector:...)`): всё ещё используется в legacy-коде
- **Не блокирует поток** — работает асинхронно через RunLoop
- **Требует явной остановки** (`invalidate()`) для повторяющихся таймеров
- **Не точен до миллисекунд** — реальное время срабатывания может отличаться на ±50–100 мс (зависит от RunLoop и нагрузки)

### Сравнение способов создания Timer (2026 стандарт)

| Способ создания                              | Преимущества                                          | Недостатки / ограничения                          | Когда использовать в 2026 |
|----------------------------------------------|-------------------------------------------------------|---------------------------------------------------|----------------------------|
| `Timer.scheduledTimer(withTimeInterval:repeats:)` с блоком | Чистый синтаксис, нет `#selector`, захват `[weak self]` | Нет `userInfo` (но можно использовать замыкание)  | **Основной и рекомендуемый** способ |
| `Timer.scheduledTimer(timeInterval:target:selector:...)` | Есть `userInfo`, совместим с Obj-C                    | Требует `@objc`, `#selector`, retain cycle риск   | Только в legacy-коде       |
| `Timer(timeInterval:repeats:)` + `RunLoop.add` | Полный контроль над RunLoop и режимом                 | Нужно вручную добавлять в RunLoop                 | Редко (специфические случаи) |

### Самые популярные и рекомендуемые паттерны 2026

#### 1. Повторяющийся таймер с блоком (самый частый)

```swift
class GameViewController: UIViewController {
    private var gameTimer: Timer?
    
    func startGameLoop() {
        gameTimer = Timer.scheduledTimer(withTimeInterval: 1.0 / 60.0, repeats: true) { [weak self] _ in
            guard let self else { return }
            self.updateGameState()
        }
    }
    
    func stopGameLoop() {
        gameTimer?.invalidate()
        gameTimer = nil
    }
    
    private func updateGameState() {
        // обновление физики, анимации и т.д. 60 раз в секунду
    }
}
```

#### 2. Однократная задержка (замена `asyncAfter`)

```swift
func showToastAfterDelay() {
    Timer.scheduledTimer(withTimeInterval: 2.0, repeats: false) { _ in
        self.showToast("Операция завершена")
    }
}
```

#### 3. Отсчёт обратного отсчёта (очень часто в UI)

```swift
class VerificationViewController: UIViewController {
    private var countdownTimer: Timer?
    private var secondsLeft = 60
    
    func startCountdown() {
        secondsLeft = 60
        countdownLabel.text = "Осталось: 60 сек"
        
        countdownTimer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [weak self] timer in
            guard let self else { return }
            
            self.secondsLeft -= 1
            self.countdownLabel.text = "Осталось: \(self.secondsLeft) сек"
            
            if self.secondsLeft <= 0 {
                timer.invalidate()
                self.resendButton.isEnabled = true
                self.countdownLabel.text = "Можно отправить повторно"
            }
        }
    }
}
```

#### 4. Таймер с userInfo (старый стиль, но иногда встречается)

```swift
let timer = Timer.scheduledTimer(timeInterval: 5.0,
                                 target: self,
                                 selector: #selector(handleTimer),
                                 userInfo: ["taskID": 42],
                                 repeats: true)

@objc func handleTimer(_ timer: Timer) {
    if let taskID = timer.userInfo as? [String: Int], let id = taskID["taskID"] {
        print("Таймер для задачи \(id)")
    }
}
```

### 5. Лучшие практики Timer в Swift 2026

- **Всегда** сохраняй ссылку на таймер (`var timer: Timer?`) и вызывай `invalidate()` при уходе с экрана / завершении работы  
- **Всегда** захватывай `[weak self]` в замыкании — предотвращает retain cycle  
- **Не используй** `Timer` для анимаций — лучше `CADisplayLink` или `UIViewPropertyAnimator`  
- **Не используй** `Timer` для очень точных интервалов (< 50 мс) — погрешность может быть большой  
- **Для SwiftUI / async** — предпочтительнее `Task { try await Task.sleep(...) }` или `.onReceive(Timer.publish(...))`  
- **Swift 6 strict concurrency** — `Timer` должен создаваться и инвалидироваться на том же акторе (обычно `@MainActor`)  
- **Документируйте** — пиши комментарий «Timer — обновление каждые 1/60 сек, invalidate в deinit / viewWillDisappear»

**Короткий итог 2026**:
> `Timer` — это «будильник в коде»: запускает код через интервалы в RunLoop.  
> В 2026 году:  
> - используй **блочную версию** с `[weak self]`  
> - всегда сохраняй и инвалидируй таймер  
> - для UI и повторяющихся задач — основной инструмент в UIKit  
> - в SwiftUI / async — чаще `Task.sleep` или `Timer.publish`  
> Это **надёжный**, но **не самый точный** способ периодических задач.

Удачи с плавными таймерами и без утечек памяти в твоём приложении! ⏲️