**viewWillTransition(to:with:)** — это метод жизненного цикла [[UIViewController]] в [[UIKit]], который вызывается **перед** изменением размеров представления контроллера (в первую очередь — при **повороте устройства**, но также при других изменениях размеров экрана).

Он появился в [[iOS]] 8 (2014) и к 2026 году остаётся **основным** и **самым надёжным** способом реагировать на смену ориентации и размеров экрана (включая iPad multitasking, split view, slide over, Dynamic Island, Stage Manager на Mac Catalyst и т.д.).

### Когда именно вызывается viewWillTransition

| Событие / Действие                            | viewWillTransition вызывается? | Передаваемые параметры | Примечание / важные детали |
|-----------------------------------------------|--------------------------------|-------------------------|-----------------------------|
| Поворот устройства (портрет ↔ ландшафт)       | **Да**                         | `newCollection`, `coordinator` | Самый частый случай |
| Изменение размеров в iPad multitasking (split view, slide over) | **Да**                         | `newCollection` содержит новый traitCollection | iPadOS 13+ |
| Stage Manager / окна на Mac Catalyst          | **Да**                         | `newCollection` отражает новый размер окна | macOS 13+ / Catalyst |
| Dynamic Island / notch / status bar изменения | **Да** (иногда)                | `newCollection` содержит новый traitCollection | iPhone 14 Pro+ |
| Изменение traitCollection (например, size class) | **Да**                         | `newCollection` — новый traitCollection | iPad / multitasking |
| Программное изменение размеров (редко)        | **Нет** (обычно)               | —                       | Если вручную вызываешь `setNeedsLayout()` — не всегда |

### Параметры метода

```swift
override func viewWillTransition(to size: CGSize, with coordinator: UIViewControllerTransitionCoordinator) {
    super.viewWillTransition(to: size, with: coordinator)
    
    // size — это **новый** размер view после перехода
    // coordinator — позволяет добавить анимации вместе с системной
}
```

### Самый популярный и рекомендуемый паттерн 2026 года

```swift
@MainActor
class AdaptiveViewController: UIViewController {
    
    private let stackView = UIStackView()
    private let imageView = UIImageView()
    private let detailLabel = UILabel()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        stackView.axis = .vertical
        stackView.spacing = 16
        stackView.translatesAutoresizingMaskIntoConstraints = false
        
        imageView.contentMode = .scaleAspectFit
        detailLabel.numberOfLines = 0
        
        stackView.addArrangedSubview(imageView)
        stackView.addArrangedSubview(detailLabel)
        view.addSubview(stackView)
        
        NSLayoutConstraint.activate([
            stackView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            stackView.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            stackView.leadingAnchor.constraint(greaterThanOrEqualTo: view.safeAreaLayoutGuide.leadingAnchor, constant: 20),
            stackView.trailingAnchor.constraint(lessThanOrEqualTo: view.safeAreaLayoutGuide.trailingAnchor, constant: -20)
        ])
    }
    
    override func viewWillTransition(to size: CGSize, with coordinator: UIViewControllerTransitionCoordinator) {
        super.viewWillTransition(to: size, with: coordinator)
        
        // 1. Определяем ориентацию / size class по новому размеру
        let isLandscape = size.width > size.height
        
        coordinator.animate(alongsideTransition: { _ in
            // 2. Анимируем изменения вместе с системной анимацией поворота
            if isLandscape {
                self.stackView.axis = .horizontal
                self.stackView.alignment = .center
                self.imageView.widthAnchor.constraint(equalToConstant: 200).isActive = true
                self.detailLabel.textAlignment = .left
            } else {
                self.stackView.axis = .vertical
                self.stackView.alignment = .fill
                self.imageView.widthAnchor.constraint(equalToConstant: 150).isActive = true
                self.detailLabel.textAlignment = .center
            }
            
            self.view.layoutIfNeeded()
        }, completion: nil)
    }
}
```

### Лучшие практики viewWillTransition в Swift 2026

- **Всегда вызывай super.viewWillTransition(to:with:)** — в начале метода  
- **Используй coordinator.animate(alongsideTransition:)** — чтобы твои анимации шли синхронно с системным поворотом  
- **Определяй ориентацию по size.width > size.height** — traitCollection не всегда точен в multitasking  
- **Не полагайся только на traitCollection** — в iPad multitasking и Stage Manager оно может не меняться  
- **@MainActor** — весь контроллер или метод — на главном акторе  
- **Swift 6 strict concurrency** — все UI-изменения — в `@MainActor`  
- **Не делай тяжёлую работу** — это место только для layout-адаптации (переключение axis, hidden/show, constraint updates)  
- **Документируйте** — пиши комментарий «viewWillTransition — адаптация layout под новую ориентацию / размер экрана»

**Короткий девиз 2026**:
> «viewWillTransition — это когда экран **вот-вот изменит размер** (поворот, split view, Stage Manager), и пора адаптировать layout: переключить ось стека, скрыть/показать элементы, обновить констрейнты.  
> Делай здесь **анимации** вместе с системными через coordinator.animate.  
> Всегда вызывай super в начале и не полагайся только на traitCollection — смотри на size.»
