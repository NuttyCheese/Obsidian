**CGSize** — это простая структура из **Core Graphics** (импортируется через `import UIKit` / `import CoreGraphics`), которая описывает **два измерения** — ширину и высоту (width и height).

Это один из самых часто используемых типов в iOS/macOS-разработке 2026 года.

### Основное определение

```swift
public struct CGSize {
    public var width: CGFloat
    public var height: CGFloat
    
    public init(width: CGFloat, height: CGFloat)
    public init()  // → CGSize.zero
}
```

- **CGFloat** — основной тип координат и размеров (на 64-битных системах — Double, на 32-битных — Float)  
- **Value type** (struct) — копируется по значению, полностью **Sendable**, безопасен в Swift Concurrency  
- **Hashable**, **Equatable**, **Codable** (с iOS 11+)

### Ключевые отличия CGSize vs CGRect vs CGPoint

| Тип      | Что описывает                  | x/y (origin) есть? | width/height есть? | Самый частый сценарий в 2026 |
|----------|--------------------------------|---------------------|---------------------|------------------------------|
| **CGSize** | Только размеры (ширина × высота) | Нет                 | **Да**              | Размер вью, изображения, ячейки, контента |
| **CGPoint** | Только позиция (координаты)     | **Да**              | Нет                 | Позиция, центр, translation жеста |
| **CGRect** | Позиция + размер (origin + size) | **Да**              | **Да**              | frame, bounds, drawing rect |

### Самые частые и важные сценарии использования CGSize в 2026

1. **Получение реального размера вью после Auto Layout**

```swift
override func viewDidLayoutSubviews() {
    super.viewDidLayoutSubviews()
    
    let viewSize: CGSize = view.bounds.size          // или view.frame.size
    let imageSize: CGSize = profileImageView.bounds.size
    
    // Делаем круглую аватарку
    profileImageView.layer.cornerRadius = min(imageSize.width, imageSize.height) / 2
}
```

2. **Установка размера для subview / layer / gradient**

```swift
gradientLayer.frame.size = CGSize(width: 300, height: 200)
// или
gradientLayer.frame = CGRect(origin: .zero, size: CGSize(width: 300, height: 200))
```

3. **Расчёт aspect ratio / scale / fitting**

```swift
let imageSize = UIImage(named: "photo")?.size ?? .zero
let targetSize = CGSize(width: 200, height: 200)

let scale = min(targetSize.width / imageSize.width, targetSize.height / imageSize.height)
let scaledSize = CGSize(width: imageSize.width * scale, height: imageSize.height * scale)

// или встроенный метод
let fittedSize = AVMakeRect(aspectRatio: imageSize, insideRect: CGRect(origin: .zero, size: targetSize)).size
```

4. **Работа с UIImage / CGImage / CALayer**

```swift
let image = UIImage(named: "avatar")
imageView.image = image?.resized(to: CGSize(width: 120, height: 120))
```

5. **В SwiftUI** — через GeometryReader или .frame

```swift
GeometryReader { geometry in
    Image("photo")
        .resizable()
        .scaledToFit()
        .frame(width: geometry.size.width * 0.8, height: geometry.size.height * 0.6)
}
```

### Полезные расширения для CGSize (очень популярны в 2026)

```swift
extension CGSize {
    // Аспект-фит / аспект-филл
    func aspectFit(in container: CGSize) -> CGSize {
        let scale = min(container.width / width, container.height / height)
        return CGSize(width: width * scale, height: height * scale)
    }
    
    func aspectFill(in container: CGSize) -> CGSize {
        let scale = max(container.width / width, container.height / height)
        return CGSize(width: width * scale, height: height * scale)
    }
    
    // Площадь
    var area: CGFloat { width * height }
    
    // Проверка пустоты
    var isEmptyOrZero: Bool { width <= 0 || height <= 0 }
    
    // Минимальный/максимальный размер
    static func +(lhs: CGSize, rhs: CGSize) -> CGSize {
        CGSize(width: lhs.width + rhs.width, height: lhs.height + rhs.height)
    }
    
    static func *(lhs: CGSize, rhs: CGFloat) -> CGSize {
        CGSize(width: lhs.width * rhs, height: lhs.height * rhs)
    }
}
```

### Лучшие практики работы с CGSize в Swift 2026

- **Читай size в viewDidLayoutSubviews() / layoutSubviews()** — до этого bounds.size обычно (0, 0)  
- **bounds.size == frame.size** — почти всегда (кроме transform/rotation)  
- **Используй safeAreaInsets / layoutMargins** — для правильных отступов  
- **@MainActor** — все операции с CGSize в UI-контексте — на главном акторе  
- **Swift 6 strict concurrency** — CGSize полностью Sendable (value type)  
- **Тестирование** — XCTAssertEqual(size1, size2, accuracy: 0.001) для floating-point  
- **Документируйте** — пиши комментарий «CGSize — актуальный размер вью после layout»

**Короткий девиз 2026**:
> «CGSize — это когда тебе нужны только **ширина и высота** без позиции.  
> В 2026 году это **основной** тип для описания размеров вью, изображений, ячеек, контента и расчёта масштаба.  
> bounds.size — твой главный источник правды о реальных размерах после Auto Layout.»

Удачи с точными размерами и адаптивным интерфейсом в Swift! 📐