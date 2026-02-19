**CGFloat** — это псевдоним типа ([[typealias]]) в **Core Graphics**, который используется практически во всех случаях, когда нужно работать с координатами, размерами, отступами, масштабами и любыми другими значениями, связанными с экраном и графикой в [[iOS]]/macOS-приложениях.

### Ключевые факты о CGFloat в 2026 году (актуально [[для]] Swift 6+)

| Платформа / Архитектура                                  | CGFloat — это  | Размер в байтах | Точность                    | Рекомендация 2026                                       |
| -------------------------------------------------------- | -------------- | --------------- | --------------------------- | ------------------------------------------------------- |
| 64-битные устройства (все современные iPhone, iPad, Mac) | **Double**     | 8 байт          | ~15–17 знаков после запятой | **Основной тип** для всего UI/графики                   |
| 32-битные устройства (очень редкие случаи, iOS < 11)     | **Float**      | 4 байта         | ~6–7 знаков                 | Почти не используется                                   |
| [[SwiftUI]] / [[UIKit]] / Core Graphics / [[Metal]]      | Всегда CGFloat | —               | —                           | Никогда не используй [[Double]]/[[Float]] напрямую в UI |

### Почему Apple использует CGFloat вместо Double/Float

1. **Единый тип для всех платформ** — один и тот же код работает и на 32-бит, и на 64-бит  
2. **Оптимизация под железо** — на 64-битных устройствах CGFloat = Double → максимальная точность  
3. **Совместимость с Core Graphics / [[Metal]]** — все [[API]] ожидают именно CGFloat  
4. **Безопасность** — исключает случайные ошибки приведения типов (CGFloat vs [[Double]])

### Самые частые сценарии использования CGFloat в 2026 году

1. **Размеры и позиционирование вью**

```swift
view.frame = CGRect(x: 20, y: 40, width: 300, height: 200)
button.center = CGPoint(x: view.bounds.midX, y: view.bounds.midY)
imageView.layer.cornerRadius = 12.0  // CGFloat
```

2. **Auto Layout constraints**

```swift
NSLayoutConstraint.activate([
    imageView.widthAnchor.constraint(equalToConstant: 120),   // CGFloat
    nameLabel.topAnchor.constraint(equalTo: imageView.bottomAnchor, constant: 16.0)
])
```

3. **SwiftUI frame / offset / padding**

```swift
Image("photo")
    .frame(width: 200, height: 200)           // CGFloat под капотом
    .offset(x: 50, y: 30)                     // CGFloat
    .padding(.horizontal, 20)                 // CGFloat
```

4. **Анимации и трансформации**

```swift
UIView.animate(withDuration: 0.3) {
    self.view.transform = CGAffineTransform(scaleX: 1.2, y: 1.2)  // CGFloat
}
```

5. **Расчёты в Core Graphics / Metal**

```swift
context.move(to: CGPoint(x: 0, y: bounds.height / 2))  // CGFloat
let path = UIBezierPath(roundedRect: bounds, cornerRadius: 16.0)
```

### Полезные расширения и константы (очень популярны в 2026)

```swift
extension CGFloat {
    var half: CGFloat { self / 2 }
    var double: CGFloat { self * 2 }
    
    static var pixel: CGFloat { 1 / UIScreen.main.scale }  // 1 физический пиксель
}

let screenWidth: CGFloat = UIScreen.main.bounds.width
let safeAreaTop: CGFloat = view.safeAreaInsets.top
```

### Лучшие практики работы с CGFloat в Swift 2026

- **Всегда используй CGFloat** — в UI, размерах, отступах, координатах, масштабах  
- **Не приводи Double/[[Float]] к CGFloat вручную** — пиши литералы как 16.0 (компилятор сам поймёт)  
- **Округляй при необходимости** — `rounded()`, `ceil()`, `floor()` для выравнивания по пикселям  
- **pixel-perfect** — используй `1 / UIScreen.main.scale` для линий толщиной в 1 физический пиксель  
- **[[@MainActor]]** — все операции с CGFloat в UI-контексте — на главном акторе  
- **Swift 6 strict concurrency** — CGFloat полностью [[Sendable]] ([[value type]])  
- **Тестирование** — XCTAssertEqual с accuracy: 0.001 (floating-point)  
- **Документируйте** — пиши комментарий «CGFloat — размер/позиция в экранных координатах»

**Короткий девиз 2026**:
> «CGFloat — это когда ты работаешь с экраном, размерами, позицией, отступами или графикой.  
> В 2026 году это **единственный правильный** тип для всего, что связано с UI и Core Graphics.  
> Никогда не используй Double/Float напрямую в размерах/координатах — только CGFloat.»
