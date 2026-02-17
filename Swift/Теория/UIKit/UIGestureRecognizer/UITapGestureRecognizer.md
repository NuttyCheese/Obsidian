**UITapGestureRecognizer** — это самый простой и самый часто используемый жест в [[UIKit]] для обнаружения **одиночного (или множественного) касания** (tap) на любом [[UIView]].

Он идеально подходит для любых действий, которые должны происходить **по нажатию**: выбор элемента, открытие деталей, toggle состояния, запуск анимации и т.д.

### Для чего используют UITapGestureRecognizer в 2026 году (реальные сценарии)

| Сценарий                                      | Количество тапов | Пример в приложениях 2026 |
|-----------------------------------------------|------------------|----------------------------|
| Одиночный тап по фото / карточке / кнопке     | 1                | Открыть полноэкранный просмотр, перейти в детали |
| Двойной тап для зума / лайка / избранного     | 2                | Зум фото в галерее, лайк в соцсетях |
| Тройной тап (редко)                           | 3                | Специальные действия в играх / редакторах |
| Тап по ячейке таблицы / коллекции             | 1                | Альтернатива didSelectRowAt / didSelectItemAt |
| Тап для показа скрытого меню / тулбара        | 1                | Показать/скрыть элементы интерфейса |
| Тап по иконке / аватарке                      | 1                | Открыть профиль пользователя |
| Тап для копирования / вставки текста          | 1                | Кастомный text selection menu |

### Что необходимо сделать, чтобы начать использовать UITapGestureRecognizer

#### Минимальный рабочий пример (2026 стиль — чистый код + @MainActor)

```swift
import UIKit

final class GalleryViewController: UIViewController {
    
    private let imageView: UIImageView = {
        let iv = UIImageView(image: UIImage(systemName: "photo.on.rectangle"))
        iv.contentMode = .scaleAspectFit
        iv.isUserInteractionEnabled = true  // обязательно!
        iv.translatesAutoresizingMaskIntoConstraints = false
        iv.backgroundColor = .systemGray6
        iv.layer.cornerRadius = 16
        return iv
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        view.backgroundColor = .systemBackground
        view.addSubview(imageView)
        
        NSLayoutConstraint.activate([
            imageView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            imageView.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            imageView.widthAnchor.constraint(equalTo: view.widthAnchor, multiplier: 0.8),
            imageView.heightAnchor.constraint(equalTo: view.heightAnchor, multiplier: 0.6)
        ])
        
        setupTapGesture()
    }
    
    private func setupTapGesture() {
        // 1. Одиночный тап — открыть полноэкранный просмотр
        let singleTap = UITapGestureRecognizer(target: self, action: #selector(handleSingleTap))
        singleTap.numberOfTapsRequired = 1
        singleTap.numberOfTouchesRequired = 1
        imageView.addGestureRecognizer(singleTap)
        
        // 2. Двойной тап — зум
        let doubleTap = UITapGestureRecognizer(target: self, action: #selector(handleDoubleTap))
        doubleTap.numberOfTapsRequired = 2
        doubleTap.numberOfTouchesRequired = 1
        imageView.addGestureRecognizer(doubleTap)
        
        // Важно: двойной тап не должен мешать одиночному
        singleTap.require(toFail: doubleTap)
    }
    
    @objc private func handleSingleTap(_ gesture: UITapGestureRecognizer) {
        // Одиночный тап → открыть детали
        let alert = UIAlertController(title: "Одиночный тап", message: "Открываем полноэкранный просмотр", preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
    
    @objc private func handleDoubleTap(_ gesture: UITapGestureRecognizer) {
        // Двойной тап → зум/отзум
        UIView.animate(withDuration: 0.3) {
            if self.imageView.transform == .identity {
                self.imageView.transform = CGAffineTransform(scaleX: 2.0, y: 2.0)
            } else {
                self.imageView.transform = .identity
            }
        }
    }
}
```

### Лучшие практики UITapGestureRecognizer в Swift 2026

- **numberOfTapsRequired** — 1 (одиночный), 2 (двойной), редко 3+  
- **numberOfTouchesRequired** — 1 (почти всегда), 2+ для мультитач  
- **require(toFail:)** — используй для разрешения конфликтов (одиночный тап ждёт, пока не завершится двойной)  
- **location(in:)** — если нужно знать точку касания (например, где именно тапнули)  
- **@objc** — обязательно для метода-обработчика  
- **[[@MainActor]]** — все обработчики жестов — на главном акторе  
- **[[Swift]] 6 strict concurrency** — [[UIGestureRecognizer]] полностью безопасен  
- **Документируйте** — пиши комментарий «UITapGestureRecognizer — одиночный тап для открытия деталей, двойной для зума»

**Короткий девиз 2026**:
> UITapGestureRecognizer — это когда тебе нужно реагировать на **одиночный или двойной тап** по вью (открыть детали, зум, toggle, меню).  
> В 2026 году это **самый популярный жест** в UIKit для любых нажатий.  
> Всегда используй require(toFail:) для двойного тапа и проверяй numberOfTapsRequired.
