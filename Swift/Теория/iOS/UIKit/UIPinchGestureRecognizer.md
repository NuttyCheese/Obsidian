**UIPinchGestureRecognizer** — это жест из [[UIKit]], который распознаёт **масштабирование двумя пальцами** (pinch in / pinch out — сжатие / растяжение).

Он используется везде, где нужно дать пользователю возможность **увеличивать/уменьшать** масштаб контента: фото, карты, PDF, графики, изображения в галерее, превью в редакторах и т.д.

### Для чего используют UIPinchGestureRecognizer в 2026 году (реальные сценарии)

| Сценарий                                      | Как именно используется                                 | Пример в приложениях 2026 |
|-----------------------------------------------|----------------------------------------------------------|----------------------------|
| Зум фото / изображений                        | Масштабирование изображения в галерее или просмотре      | Фото в Instagram / Photos.app |
| Масштабирование карты / PDF / документов      | Пинч для приближения/удаления                           | Apple Maps, PDF Reader, Notes |
| Зум контента в коллекциях / таблицах          | Увеличение ячеек или превью                             | Галерея товаров в e-commerce |
| Редактирование изображений / графики          | Масштабирование холста или слоя                         | Фоторедакторы, Procreate-подобные |
| Интерактивные графики / диаграммы             | Зум области данных                                      | Финансовые приложения, аналитика |
| Просмотр 3D-моделей / AR-контента             | Масштабирование объекта в превью                       | AR Quick Look, 3D Viewer |

### Что необходимо сделать, чтобы начать использовать UIPinchGestureRecognizer

#### Минимальный рабочий пример (2026 стиль — чистый код + @MainActor)

```swift
import UIKit

final class ImageZoomViewController: UIViewController {
    
    private let imageView: UIImageView = {
        let iv = UIImageView(image: UIImage(named: "sample_photo"))
        iv.contentMode = .scaleAspectFit
        iv.isUserInteractionEnabled = true
        iv.translatesAutoresizingMaskIntoConstraints = false
        return iv
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        view.backgroundColor = .black
        view.addSubview(imageView)
        
        NSLayoutConstraint.activate([
            imageView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            imageView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            imageView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            imageView.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor)
        ])
        
        // 1. Создаём жест масштабирования
        let pinch = UIPinchGestureRecognizer(target: self, action: #selector(handlePinch))
        
        // 2. Настраиваем (опционально)
        pinch.numberOfTouchesRequired = 2           // строго двумя пальцами
        pinch.maximumNumberOfTouches = 2
        
        // 3. Добавляем жест к imageView
        imageView.addGestureRecognizer(pinch)
    }
    
    @objc private func handlePinch(_ gesture: UIPinchGestureRecognizer) {
        guard let view = gesture.view else { return }
        
        switch gesture.state {
        case .began, .changed:
            // Применяем текущий масштаб
            view.transform = view.transform.scaledBy(x: gesture.scale, y: gesture.scale)
            
            // Сбрасываем scale для следующего события .changed
            gesture.scale = 1.0
            
        case .ended, .cancelled:
            // Опционально: анимируем возврат к нормальному масштабу, если слишком маленький/большой
            let currentScale = view.transform.a  // масштаб по X
            if currentScale < 0.5 || currentScale > 4.0 {
                UIView.animate(withDuration: 0.3) {
                    view.transform = .identity
                }
            }
            
        default:
            break
        }
    }
}
```

### Лучшие практики UIPinchGestureRecognizer в Swift 2026

- **numberOfTouchesRequired = 2** — почти всегда (чтобы не мешал одиночным жестам)  
- **gesture.scale = 1.0** — **обязательно** сбрасывай после обработки .changed (иначе масштаб накопится)  
- **transform.scaledBy** — самый простой и плавный способ масштабирования  
- **Ограничение масштаба** — проверяй в .ended (например, min 0.5, max 4.0) и анимируй возврат  
- **[[@objc]]** — обязательно для метода-обработчика  
- **[[@MainActor]]** — все обработчики жестов — на главном акторе  
- **[[Swift]] 6 strict concurrency** — [[UIGestureRecognizer]] полностью безопасен  
- **Документируйте** — пиши комментарий «UIPinchGestureRecognizer — зум изображения двумя пальцами»

**Короткий девиз 2026**:
> UIPinchGestureRecognizer — это когда тебе нужно дать пользователю возможность **масштабировать контент двумя пальцами** (pinch in/out).  
> В 2026 году это **основной жест** для зума фото, карт, документов, превью и графики.  
> Всегда сбрасывай gesture.scale = 1.0 в .changed и проверяй state.
