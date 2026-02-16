**CADisplayLink** — это низкоуровневый механизм синхронизации анимаций и рендеринга с частотой обновления экрана в [[iOS]], iPadOS, tvOS и macOS Catalyst (через [[UIKit]]/AppKit).

По состоянию на февраль 2026 года CADisplayLink остаётся **самым точным и энергоэффективным** способом привязать код к рефрешу экрана (обычно 60, 80, 90, 120 Гц), особенно когда нужно делать что-то **каждый кадр**.

### Когда использовать CADisplayLink в 2026 году (реальные сценарии)

| Сценарий                                         | Почему CADisplayLink всё ещё лучший выбор   | Современная альтернатива (если есть) |
| ------------------------------------------------ | ------------------------------------------- | ------------------------------------ |
| Кастомные анимации с физикой (частицы, игра)     | Точная привязка к кадру, минимальный jitter | Metal / SceneKit / SpriteKit         |
| Реал-тайм рендеринг графики (OpenGL / [[Metal]]) | Синхронизация с V-Sync                      | Metal + CAMetalLayer                 |
| Обновление UI каждые 1/120 сек (очень плавно)    | Самый низкий input lag                      | CADisplayLink + CATransaction        |
| Измерение FPS / профилирование рендеринга        | Самый точный способ узнать реальный кадр    | Instruments + os_signpost            |
| Legacy-код / старые OpenGL-проекты               | Полная совместимость                        | —                                    |

**Не используй**, если:

- Нужна простая анимация → **[[UIView]].animate**, **CAAnimation**, **[[SwiftUI]] Animation**  
- Нужен declarative UI → **SwiftUI .onReceive** / **TimelineView**  
- Нужен Metal-рендеринг → **CAMetalLayer + CVDisplayLink** (macOS) или **MTKView**  

### Как правильно использовать CADisplayLink в 2026 году

#### Самый современный и безопасный паттерн

```swift
import QuartzCore

final class SmoothAnimator {
    
    private var displayLink: CADisplayLink?
    private weak var target: AnyObject?
    private let selector: Selector
    
    init(target: AnyObject, selector: Selector) {
        self.target = target
        self.selector = selector
    }
    
    func start() {
        guard displayLink == nil else { return }
        
        // Самый важный параметр — preferredFramesPerSecond
        displayLink = CADisplayLink(target: target!, selector: selector)
        displayLink?.preferredFramesPerSecond = 120  // или 0 = максимум
        displayLink?.isPaused = false
        
        // Добавляем в RunLoop.current (важно!)
        displayLink?.add(to: .current, forMode: .default)
    }
    
    func pause() {
        displayLink?.isPaused = true
    }
    
    func stop() {
        displayLink?.invalidate()
        displayLink = nil
    }
    
    deinit {
        stop()
    }
}

// Пример использования в UIViewController
class GameViewController: UIViewController {
    
    private lazy var animator = SmoothAnimator(target: self, selector: #selector(updateFrame))
    
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        animator.start()
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        animator.stop()
    }
    
    @objc private func updateFrame(displayLink: CADisplayLink) {
        // Здесь происходит рендеринг каждого кадра
        let timestamp = displayLink.targetTimestamp
        let duration = displayLink.duration          // ~0.00833 при 120 Гц
        let fps = 1.0 / duration
        
        // Пример: физика частиц
        updateParticles(deltaTime: duration)
        
        // Можно логировать FPS для отладки
        print("Frame time: \(duration * 1000) ms, FPS: \(fps)")
    }
    
    private func updateParticles(deltaTime: CFTimeInterval) {
        // ваша логика
    }
}
```

### Ключевые свойства CADisplayLink (самые полезные в 2026)

| Свойство                     | Тип                  | Что возвращает / делает                              | Рекомендация 2026 |
|------------------------------|----------------------|-------------------------------------------------------|-------------------|
| `targetTimestamp`            | CFTimeInterval       | Время, когда кадр **должен** отобразиться             | Используй для физики |
| `timestamp`                  | CFTimeInterval       | Время последнего вызова callback                     | Для delta-time |
| `duration`                   | CFTimeInterval       | Время между кадрами (~1/60, 1/80, 1/90, 1/120 сек)   | Для расчёта FPS |
| `preferredFramesPerSecond`   | Int                  | Желаемая частота (0 = максимум, 30, 60, 120 и т.д.)  | Устанавливай 120 на ProMotion |
| `isPaused`                   | Bool                 | Приостановка без invalidate                           | Для паузы игры |
| `add(to:forMode:)`           | —                    | Добавление в RunLoop (`.current`, `.default`)        | Обязательно! |

### Лучшие практики CADisplayLink в Swift 2026

- **Никогда** не держи сильную ссылку на CADisplayLink → используй `weak` target  
- **invalidate()** в `deinit` / `viewWillDisappear` — обязательно  
- **preferredFramesPerSecond = 120** — на ProMotion-дисплеях (iPhone 13 Pro+, iPad Pro, MacBook Pro 2021+)  
- **Не делай тяжёлую работу в callback** — максимум 1–2 мс, иначе дропы кадров  
- **@MainActor** — почти всегда, т.к. CADisplayLink работает на main run loop  
- **Swift 6 strict concurrency** — используй `nonisolated(unsafe)` только если callback не касается UI  
- **Тестирование** — сложно тестировать → чаще используют snapshot или manual проверку FPS в Instruments  
- **Документируйте** — пишите комментарий «CADisplayLink — рендеринг каждого кадра, <1 мс»

**Короткий девиз 2026**:
> «CADisplayLink — это когда тебе нужно идеально синхронизировать код с каждым кадром экрана (60/120 Гц).  
> В 2026 году это **лучший выбор** для кастомной физики, частиц, игр и реал-тайм графики в UIKit/AppKit.  
> Для простых анимаций — используй SwiftUI Animation или CAAnimation.»
