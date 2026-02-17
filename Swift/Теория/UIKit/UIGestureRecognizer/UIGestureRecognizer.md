**UIGestureRecognizer** — это базовый класс в UIKit, который отвечает за **распознавание жестов** пользователя (тап, свайп, пинч, лонг-пресс, пан и т.д.).

Он позволяет легко добавлять интерактивность к любому `UIView` (в том числе к `UITableViewCell`, `UICollectionViewCell`, `UIImageView`, `UIView` и т.д.).

### Для чего используют UIGestureRecognizer в 2026 году (реальные сценарии)

| Жест / Класс                              | Основное назначение в 2026 году                     | Самый частый пример использования |
|-------------------------------------------|------------------------------------------------------|-----------------------------------|
| **UITapGestureRecognizer**                | Одиночный / двойной тап                             | Открыть фото, выбрать элемент, toggle switch |
| **UILongPressGestureRecognizer**          | Долгое нажатие                                      | Контекстное меню, drag & drop, редактирование |
| **UIPanGestureRecognizer**                | Перетаскивание пальцем                              | Перемещение вью, pull-to-refresh, свайп-меню |
| **UISwipeGestureRecognizer**              | Быстрый свайп в сторону                             | Swipe-to-delete, свайп для перехода |
| **UIPinchGestureRecognizer**              | Масштабирование (пинч двумя пальцами)               | Зум фото/карты/контента |
| **UIRotationGestureRecognizer**           | Вращение двумя пальцами                             | Поворот фото/объекта в редакторе |
| **UIScreenEdgePanGestureRecognizer**      | Свайп с края экрана                                 | Переход назад (как в iOS навигации) |
| **Комбинации жестов**                     | Одновременное использование нескольких              | Пинч + пан = зум + перемещение |

### Что необходимо сделать, чтобы начать использовать UIGestureRecognizer

#### Минимальный рабочий пример (2026 стиль)

```swift
import UIKit

class ImageViewController: UIViewController {
    
    private let imageView = UIImageView(image: UIImage(systemName: "photo"))
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        imageView.contentMode = .scaleAspectFit
        imageView.isUserInteractionEnabled = true  // обязательно!
        imageView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(imageView)
        
        NSLayoutConstraint.activate([
            imageView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            imageView.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            imageView.widthAnchor.constraint(equalTo: view.widthAnchor, multiplier: 0.8),
            imageView.heightAnchor.constraint(equalTo: view.heightAnchor, multiplier: 0.6)
        ])
        
        setupGestures()
    }
    
    private func setupGestures() {
        // 1. Одиночный тап — показать алерт
        let tap = UITapGestureRecognizer(target: self, action: #selector(handleTap))
        imageView.addGestureRecognizer(tap)
        
        // 2. Двойной тап — зум
        let doubleTap = UITapGestureRecognizer(target: self, action: #selector(handleDoubleTap))
        doubleTap.numberOfTapsRequired = 2
        imageView.addGestureRecognizer(doubleTap)
        
        // 3. Долгое нажатие — показать меню
        let longPress = UILongPressGestureRecognizer(target: self, action: #selector(handleLongPress))
        longPress.minimumPressDuration = 0.5
        imageView.addGestureRecognizer(longPress)
        
        // 4. Пинч (масштабирование)
        let pinch = UIPinchGestureRecognizer(target: self, action: #selector(handlePinch))
        imageView.addGestureRecognizer(pinch)
        
        // 5. Пан (перетаскивание)
        let pan = UIPanGestureRecognizer(target: self, action: #selector(handlePan))
        imageView.addGestureRecognizer(pan)
        
        // Важно: двойной тап не должен мешать одиночному
        tap.require(toFail: doubleTap)
    }
    
    @objc private func handleTap(_ gesture: UITapGestureRecognizer) {
        let location = gesture.location(in: imageView)
        print("Тап в точке: \(location)")
        // Например, показать алерт или зум
    }
    
    @objc private func handleDoubleTap(_ gesture: UITapGestureRecognizer) {
        // Зум при двойном тапе
        UIView.animate(withDuration: 0.3) {
            self.imageView.transform = self.imageView.transform == .identity 
                ? CGAffineTransform(scaleX: 2, y: 2) 
                : .identity
        }
    }
    
    @objc private func handleLongPress(_ gesture: UILongPressGestureRecognizer) {
        if gesture.state == .began {
            // Показать контекстное меню
            let alert = UIAlertController(title: "Действия", message: nil, preferredStyle: .actionSheet)
            alert.addAction(UIAlertAction(title: "Сохранить", style: .default))
            alert.addAction(UIAlertAction(title: "Отмена", style: .cancel))
            present(alert, animated: true)
        }
    }
    
    @objc private func handlePinch(_ gesture: UIPinchGestureRecognizer) {
        imageView.transform = imageView.transform.scaledBy(x: gesture.scale, y: gesture.scale)
        gesture.scale = 1.0
    }
    
    @objc private func handlePan(_ gesture: UIPanGestureRecognizer) {
        let translation = gesture.translation(in: view)
        imageView.center = CGPoint(x: imageView.center.x + translation.x, 
                                  y: imageView.center.y + translation.y)
        gesture.setTranslation(.zero, in: view)
    }
}
```

### Лучшие практики UIGestureRecognizer в Swift 2026

- **isUserInteractionEnabled = true** — обязательно для вью, к которой добавляешь жест  
- **addGestureRecognizer** — добавляй жесты к нужной вью (не к контроллеру)  
- **require(toFail:)** — используй для разрешения конфликтов (двойной тап → требует одиночного)  
- **@objc** — обязательно для методов, которые передаёшь в селектор  
- **gesture.state** — проверяй .began / .changed / .ended / .cancelled в обработчиках  
- **@MainActor** — все обработчики жестов — на главном акторе  
- **Swift 6 strict concurrency** — UIGestureRecognizer полностью безопасен  
- **Документируйте** — пиши комментарий «UIGestureRecognizer — тап и лонг-пресс для открытия контекстного меню»

**Короткий девиз 2026**:
> UIGestureRecognizer — это когда ты хочешь добавить **интерактивность** любому вью: тап, свайп, пинч, лонг-пресс, пан и т.д.  
> В 2026 году это **основной инструмент** для жестов в UIKit.  
> Добавляй жест → пиши @objc-метод → используй require(toFail:) для разрешения конфликтов.

Удачи с отзывчивым и интуитивным интерфейсом в Swift! 👆