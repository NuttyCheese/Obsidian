**CALayer** — это **низкоуровневый объект** из Core Animation, который управляет **всем визуальным содержимым и анимациями** любого `UIView`.

Каждый `UIView` имеет **свой собственный слой** (`view.layer`), и именно через этот слой происходят:
- отрисовка фона, текста, изображений
- скругление углов, границы, тени
- маски, градиенты, маскирование
- все анимации (перемещение, масштаб, прозрачность, вращение и т.д.)

Проще говоря:  
`UIView` — это «логический контейнер» (обработка событий, layout, иерархия),  
а `CALayer` — это «визуальный движок», который рисует всё на экране.

### Почему CALayer важен в 2026 году

| Возможность                          | Через UIView (старый способ)                  | Через CALayer (современный способ)                  | Почему CALayer лучше |
|--------------------------------------|------------------------------------------------|-----------------------------------------------------|----------------------|
| Скругление углов                     | `layer.cornerRadius`                           | `layer.cornerRadius`                                | То же, но быстрее и точнее |
| Тень                                 | `layer.shadow*` свойства                       | `layer.shadow*` + `shadowPath`                      | Тень без `shadowPath` очень дорогая |
| Градиент / маска                     | Нужно вручную рисовать в `draw(_:)`           | `CAGradientLayer`, `mask`                           | Гораздо проще и быстрее |
| Анимации                             | `UIView.animate`                               | `CABasicAnimation`, `CAKeyframeAnimation`           | Полный контроль, поддержка completion, timing |
| Сложные композиции                   | Много вложенных вью                            | Иерархия `sublayers`                                | Меньше overhead, лучше производительность |
| 3D-трансформации                     | Ограничено                                     | `transform`, `sublayerTransform`, perspective       | Полноценная 3D-графика |

### Основные свойства CALayer (самые используемые в 2026)

| Свойство / Метод                     | Тип                              | Что делает / Пример использования 2026 |
|--------------------------------------|----------------------------------|----------------------------------------|
| `cornerRadius`                       | CGFloat                          | Скругление углов (самое частое)        |
| `borderWidth` / `borderColor`        | CGFloat / CGColor                | Тонкая рамка вокруг карточек           |
| `shadowOpacity` / `shadowRadius` / `shadowOffset` / `shadowColor` | Float / CGFloat / CGSize / CGColor | Тень (обязательно используй `shadowPath`) |
| `shadowPath`                         | CGPath?                          | Оптимизация тени (экономит GPU)        |
| `masksToBounds`                      | Bool                             | Обрезать содержимое по границам слоя   |
| `backgroundColor`                    | CGColor?                         | Цвет фона (чаще используют view.backgroundColor) |
| `contents`                           | Any? (обычно CGImage)            | Прямое изображение без UIImageView     |
| `sublayers`                          | [CALayer]?                       | Добавление градиентов, масок, эффектов |
| `addSublayer(_:)`                    | —                                | Добавление подслоя (градиент, маска)   |
| `add(_:forKey:)`                     | —                                | Добавление анимации                    |

### Самый популярный и современный паттерн 2026 года

```swift
final class CardView: UIView {
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupLayer()
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    private func setupLayer() {
        // Основные визуальные свойства
        backgroundColor = .systemBackground
        layer.cornerRadius = 16
        layer.masksToBounds = false  // важно для тени!
        
        // Тень (самый частый и дорогой эффект)
        layer.shadowColor = UIColor.black.cgColor
        layer.shadowOpacity = 0.25
        layer.shadowOffset = CGSize(width: 0, height: 4)
        layer.shadowRadius = 12
        
        // Оптимизация тени (обязательно!)
        layer.shadowPath = UIBezierPath(roundedRect: bounds, cornerRadius: layer.cornerRadius).cgPath
        
        // Пример подслоя — градиент
        let gradientLayer = CAGradientLayer()
        gradientLayer.frame = bounds
        gradientLayer.colors = [UIColor.systemBlue.cgColor, UIColor.systemPurple.cgColor]
        gradientLayer.startPoint = CGPoint(x: 0, y: 0)
        gradientLayer.endPoint = CGPoint(x: 1, y: 1)
        layer.insertSublayer(gradientLayer, at: 0)
    }
    
    // Обновляем shadowPath при изменении размеров
    override func layoutSubviews() {
        super.layoutSubviews()
        layer.shadowPath = UIBezierPath(roundedRect: bounds, cornerRadius: layer.cornerRadius).cgPath
        
        // Обновляем градиент (если он есть)
        if let gradient = layer.sublayers?.first as? CAGradientLayer {
            gradient.frame = bounds
        }
    }
}
```

### Лучшие практики работы с CALayer в Swift 2026

- **Всегда используй `shadowPath`** — без него тень очень дорогая по GPU  
- **masksToBounds = false** — если есть тень (иначе тень обрезается)  
- **layoutSubviews()** — обновляй `shadowPath`, `frame` подслоёв здесь  
- **CAGradientLayer**, `CAShapeLayer`, `CAReplicatorLayer` — добавляй как sublayers для сложных эффектов  
- **CABasicAnimation** / **CAKeyframeAnimation** — для сложных анимаций (UIView.animate не всегда хватает)  
- **@MainActor** — все операции с CALayer — на главном акторе  
- **Swift 6 strict concurrency** — CALayer полностью безопасен  
- **Документируйте** — пиши комментарий «CALayer — скругление + оптимизированная тень + градиент»

**Короткий девиз 2026**:
> CALayer — это «визуальный движок» под каждым UIView.  
> Скругление, тень, градиент, маска, анимация — всё это делается **только** через layer.  
> В 2026 году: shadowPath обязательно, sublayers для эффектов, layoutSubviews для обновлений.

Удачи с красивым и производительным UI в Swift! ✨