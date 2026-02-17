**UIPanGestureRecognizer** — это жест из UIKit, который распознаёт **перемещение пальца(ев)** по экрану (pan / drag / свайп с удержанием).

Он идеально подходит для всего, что связано с **перетаскиванием**, **скроллом вручную**, **перемещением объектов**, **свайп-меню** и т.д.

### Для чего используют UIPanGestureRecognizer в 2026 году (реальные сценарии)

| Сценарий                                      | Как именно используется                                 | Пример в приложениях 2026 |
|-----------------------------------------------|----------------------------------------------------------|----------------------------|
| Перетаскивание карточек / задач / виджетов    | Изменение center или transform в процессе .changed      | Trello-подобные канбан-доски, перестановка задач |
| Кастомный pull-to-refresh                     | Отслеживание translation.y для анимации pull             | Кастомные рефреш-контролы |
| Горизонтальный / вертикальный свайп-меню     | Определение направления и расстояния свайпа             | Swipe-to-delete, reveal actions |
| Перемещение вью по экрану (draggable view)    | Обновление center в реальном времени                    | Перетаскиваемые стикеры, поповеры, floating buttons |
| Жест "смахнуть для перехода назад"            | Свайп слева направо → navigationController.popViewController | Кастомный back gesture |
| Параллакс-эффект при скролле                  | Связка pan + scrollView.contentOffset                   | Параллакс-хедеры в профилях |
| Ручной скролл внутри ScrollView               | Перехват pan для кастомной прокрутки                    | Кастомные карусели, infinite scroll |

### Что необходимо сделать, чтобы начать использовать UIPanGestureRecognizer

#### Минимальный рабочий пример (2026 стиль — чистый код + @MainActor)

```swift
import UIKit

final class DraggableViewController: UIViewController {
    
    private let draggableView: UIView = {
        let v = UIView()
        v.backgroundColor = .systemBlue
        v.layer.cornerRadius = 16
        v.translatesAutoresizingMaskIntoConstraints = false
        return v
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        view.backgroundColor = .systemBackground
        view.addSubview(draggableView)
        
        NSLayoutConstraint.activate([
            draggableView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            draggableView.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            draggableView.widthAnchor.constraint(equalToConstant: 120),
            draggableView.heightAnchor.constraint(equalToConstant: 120)
        ])
        
        // 1. Создаём жест перетаскивания
        let pan = UIPanGestureRecognizer(target: self, action: #selector(handlePan))
        
        // 2. Настраиваем (опционально)
        pan.maximumNumberOfTouches = 1          // только одним пальцем
        pan.minimumNumberOfTouches = 1
        
        // 3. Добавляем жест к нужной вью
        draggableView.addGestureRecognizer(pan)
        
        // Важно!
        draggableView.isUserInteractionEnabled = true
    }
    
    @objc private func handlePan(_ gesture: UIPanGestureRecognizer) {
        guard let view = gesture.view else { return }
        
        // Получаем текущее перемещение относительно начальной точки
        let translation = gesture.translation(in: view.superview)
        
        switch gesture.state {
        case .began:
            // Начало перетаскивания — можно сохранить начальную позицию
            print("Начало перетаскивания")
            
        case .changed:
            // Перемещаем вью
            view.center = CGPoint(
                x: view.center.x + translation.x,
                y: view.center.y + translation.y
            )
            
            // Сбрасываем translation для следующего события .changed
            gesture.setTranslation(.zero, in: view.superview)
            
        case .ended, .cancelled:
            // Завершение — можно добавить анимацию возврата или snap
            UIView.animate(withDuration: 0.3) {
                // Например, вернуть в центр, если ушло далеко
                if view.frame.midX < self.view.bounds.midX - 100 {
                    view.center.x = self.view.bounds.midX - 150
                }
            }
            
        default:
            break
        }
    }
}
```

### Лучшие практики UIPanGestureRecognizer в Swift 2026

- **isUserInteractionEnabled = true** — обязательно для вью, к которой добавляешь жест  
- **translation(in:)** — передавай superview или view, относительно которой считаешь перемещение  
- **setTranslation(.zero, in:)** — обязательно сбрасывай после обработки .changed (иначе накопится смещение)  
- **gesture.state** — всегда проверяй .began / .changed / .ended / .cancelled  
- **@objc** — обязательно для метода-обработчика  
- **@MainActor** — все обработчики жестов — на главном акторе  
- **Swift 6 strict concurrency** — UIGestureRecognizer полностью безопасен  
- **Документируйте** — пиши комментарий «UIPanGestureRecognizer — перетаскивание карточки по экрану»

**Короткий девиз 2026**:
> UIPanGestureRecognizer — это когда тебе нужно **перетаскивать, двигать, свайпать или тянуть** вью по экрану в реальном времени.  
> В 2026 году это **основной жест** для drag & drop, кастомных свайпов, pull-to-refresh и интерактивных элементов.  
> Всегда сбрасывай translation в .changed и проверяй state.

Удачи с плавным и интуитивным перетаскиванием в Swift! 👆