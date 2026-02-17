**UIRotationGestureRecognizer** — это жест из [[UIKit]], который распознаёт **вращение двумя пальцами** (rotation gesture) на любом [[UIView]].

Он используется, когда нужно дать пользователю возможность **поворачивать** объекты: фото, стикеры, изображения в редакторах, 3D-модели в превью, элементы интерфейса и т.д.

### Для чего используют UIRotationGestureRecognizer в 2026 году (реальные сценарии)

| Сценарий                                      | Как именно используется                                 | Пример в приложениях 2026 |
|-----------------------------------------------|----------------------------------------------------------|----------------------------|
| Поворот фото / стикеров в редакторе           | Применение rotation к transform в .changed               | Фоторедакторы (PicsArt, Photoshop Express, Markup) |
| Вращение объектов в AR / 3D-превью            | Поворот модели или объекта в пространстве               | AR Quick Look, 3D Viewer, мебель в AR |
| Редактирование графики / иллюстраций          | Поворот слоя или холста                                 | Procreate, Affinity Designer, Vectornator |
| Интерактивные элементы (карточки, виджеты)    | Поворот для кастомных анимаций или игр                  | Кастомные стикеры в чатах, интерактивные карточки |
| Просмотр 360° фото / панорам                  | Вращение для изменения угла обзора                      | 360°-фото в галереях, VR-превью |
| Игры / креативные приложения                  | Поворот объектов в 2D/3D-пространстве                   | Пазлы, конструкторы, drawing apps |

### Что необходимо сделать, чтобы начать использовать UIRotationGestureRecognizer

#### Минимальный рабочий пример (2026 стиль — чистый код + @MainActor)

```swift
import UIKit

final class RotationViewController: UIViewController {
    
    private let rotatableView: UIView = {
        let v = UIView()
        v.backgroundColor = .systemOrange
        v.layer.cornerRadius = 20
        v.translatesAutoresizingMaskIntoConstraints = false
        return v
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        view.backgroundColor = .systemBackground
        view.addSubview(rotatableView)
        
        NSLayoutConstraint.activate([
            rotatableView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            rotatableView.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            rotatableView.widthAnchor.constraint(equalToConstant: 200),
            rotatableView.heightAnchor.constraint(equalToConstant: 200)
        ])
        
        // 1. Создаём жест вращения
        let rotation = UIRotationGestureRecognizer(target: self, action: #selector(handleRotation))
        
        // 2. Настраиваем (опционально)
        rotation.numberOfTouchesRequired = 2          // строго двумя пальцами
        rotation.maximumNumberOfTouches = 2
        
        // 3. Добавляем жест к нужной вью
        rotatableView.addGestureRecognizer(rotation)
        
        // Важно!
        rotatableView.isUserInteractionEnabled = true
    }
    
    @objc private func handleRotation(_ gesture: UIRotationGestureRecognizer) {
        guard let view = gesture.view else { return }
        
        switch gesture.state {
        case .began, .changed:
            // Применяем текущее вращение
            view.transform = view.transform.rotated(by: gesture.rotation)
            
            // Сбрасываем rotation для следующего события .changed
            gesture.rotation = 0
            
        case .ended, .cancelled:
            // Опционально: анимируем "примагничивание" к 90° шагам
            let currentAngle = atan2(view.transform.b, view.transform.a)
            let snappedAngle = round(currentAngle / .pi * 2) * .pi / 2  // ближайшие 90°
            
            UIView.animate(withDuration: 0.3) {
                view.transform = CGAffineTransform(rotationAngle: snappedAngle)
            }
            
        default:
            break
        }
    }
}
```

### Лучшие практики UIRotationGestureRecognizer в Swift 2026

- **numberOfTouchesRequired = 2** — почти всегда (чтобы не мешал одиночным жестам)  
- **gesture.rotation = 0** — **обязательно** сбрасывай после обработки .changed (иначе вращение накопится)  
- **transform.rotated(by:)** — самый простой и плавный способ поворота  
- **Ограничение угла** — проверяй в .ended (например, snap to 90° или лимит ±180°) и анимируй возврат  
- **[[@objc]]** — обязательно для метода-обработчика  
- **[[@MainActor]]** — все обработчики жестов — на главном акторе  
- **[[Swift]] 6 strict concurrency** — UIGestureRecognizer полностью безопасен  
- **Документируйте** — пиши комментарий «UIRotationGestureRecognizer — поворот объекта двумя пальцами с snap на 90°»

**Короткий девиз 2026**:
> UIRotationGestureRecognizer — это когда тебе нужно дать пользователю возможность **поворачивать объект двумя пальцами** (rotation gesture).  
> В 2026 году это **основной жест** для редактирования фото, стикеров, 3D-превью и интерактивных элементов.  
> Всегда сбрасывай gesture.rotation = 0 в .changed и проверяй state.
