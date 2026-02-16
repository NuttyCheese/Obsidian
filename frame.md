**bounds** в [[UIKit]]/[[AppKit]]/[[SwiftUI]] — это свойство типа **[[CGRect]]**, которое описывает **размеры и положение** вью (или контроллера) **в её собственной системе координат** (относительно её собственного origin, который всегда (0, 0)).

### Ключевые отличия bounds vs frame (самое важное в 2026 году)

| Свойство       | Что возвращает                                    | origin всегда (0, 0)? | Меняется ли при transform/rotation?  | Когда использовать в 2026                            |
| -------------- | ------------------------------------------------- | --------------------- | ------------------------------------ | ---------------------------------------------------- |
| **[[bounds]]** | CGRect с origin = (0, 0) и size = ширина × высота | **Да**                | **Нет** (transform не влияет)        | Когда нужны **реальные размеры** вью (ширина/высота) |
| **frame**      | CGRect в системе координат **супервида**          | **Нет**               | **Да** (сдвигается и поворачивается) | Когда нужна **позиция** относительно родителя        |

### Самые частые и важные сценарии использования bounds в 2026

1. **Получение актуальных размеров после Auto Layout**  
   Самый частый случай — именно в `viewDidLayoutSubviews()` или `layoutSubviews()`

```swift
override func viewDidLayoutSubviews() {
    super.viewDidLayoutSubviews()
    
    // bounds уже содержит актуальные размеры после Auto Layout
    let width = bounds.width
    let height = bounds.height
    
    // Например, круглая аватарка
    profileImageView.layer.cornerRadius = bounds.width / 2  // или height / 2
    
    // Градиент на всю вью
    gradientLayer.frame = bounds
}
```

2. **Создание shadow path / mask / corner radius**  
   bounds — это **единственный надёжный** источник размеров

```swift
profileImageView.layer.shadowPath = UIBezierPath(roundedRect: profileImageView.bounds, cornerRadius: 60).cgPath
```

3. **Расчёт внутренних отступов / content insets**

```swift
scrollView.contentInset = UIEdgeInsets(top: 0, left: 0, bottom: bounds.height * 0.1, right: 0)
```

4. **Проверка, изменились ли размеры** (часто в [[viewDidLayoutSubviews]])

```swift
private var previousBoundsSize: CGSize = .zero

override func viewDidLayoutSubviews() {
    super.viewDidLayoutSubviews()
    
    if bounds.size != previousBoundsSize {
        previousBoundsSize = bounds.size
        updateLayoutForNewSize()
    }
}
```

5. **В SwiftUI** — bounds доступен через `GeometryReader`

```swift
GeometryReader { geometry in
    Text("Ширина экрана: \(geometry.size.width)")
        .frame(maxWidth: .infinity, maxHeight: .infinity)
}
```

### Лучшие практики работы с bounds в Swift 2026

- **Никогда не читай bounds в [[viewDidLoad]]()** — там размеры обычно (0, 0) или неверные  
- **Читай bounds в viewDidLayoutSubviews() / layoutSubviews()** — это **единственное гарантированно правильное** место  
- **bounds.origin всегда (0, 0)** — не пытайся использовать его для позиционирования  
- **Используй safeAreaLayoutGuide / readableContentGuide** — вместо bounds для отступов от краёв экрана  
- **@MainActor** — все операции с bounds/frame — на главном акторе  
- **Swift 6 strict concurrency** — bounds — [[Sendable]]-safe (CGRect — value type)  
- **Не меняй bounds напрямую** — это приведёт к undefined behavior; меняй frame или constraints  
- **Документируйте** — пиши комментарий «bounds — актуальные размеры после Auto Layout»

**Короткий девиз 2026**:
> «bounds — это когда тебе нужны **реальные внутренние размеры** вью после всей компоновки Auto Layout и transform’ов.  
> origin всегда (0, 0), size — ширина × высота.  
> Читай bounds **только** в viewDidLayoutSubviews / [[layoutSubviews]].  
> frame — для позиции относительно супервида, bounds — для размеров внутри себя.»
