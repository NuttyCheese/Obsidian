**subviews** в [[UIKit]] — это массив всех **прямых дочерних представлений** (children views) текущего [[UIView]].  

Это свойство типа `[UIView]` и оно **только для чтения** — ты не можешь напрямую присвоить ему новый массив.  
Чтобы изменить содержимое, используй методы:

- `addSubview(_:)`
- `insertSubview(_:at:)`
- `insertSubview(_:aboveSubview:)`
- `insertSubview(_:belowSubview:)`
- `bringSubviewToFront(_:)`
- `sendSubviewToBack(_:)`
- `removeFromSuperview()`
- `exchangeSubview(at:at:)`

### Ключевые характеристики subviews (2026 актуально)

| Свойство / Поведение                        | Что это значит                                                                 | Важно помнить |
|---------------------------------------------|--------------------------------------------------------------------------------|---------------|
| **Только прямые дети**                      | subviews содержит **только** те вью, которые добавлены напрямую через addSubview | Внуки (subviews субвью) не попадают в массив |
| **Порядок = z-order**                       | Порядок элементов в массиве определяет порядок отрисовки (позже добавленные — выше) | Первый элемент — самый нижний слой, последний — самый верхний |
| **subviews.reversed()**                     | Часто используется в hitTest и drawing, чтобы идти сверху вниз                 | Самый верхний слой проверяется первым |
| **Изменяется автоматически**                | При вызове addSubview/removeFromSuperview массив обновляется сам               | Не нужно вручную синхронизировать |
| **Не копируется**                           | subviews — это **живая** ссылка на внутренний массив, а не копия              | let subs = view.subviews — это тот же массив |
| **Thread-safe только на main**              | Все операции с subviews должны быть на главном потоке (@MainActor)             | В Swift 6 это строго проверяется |

### Самые частые и полезные сценарии использования subviews

#### 1. Перебор всех дочерних вью (поиск, очистка, анимация)

```swift
// Скрыть все UILabel внутри вью
for subview in contentView.subviews {
    if let label = subview as? UILabel {
        label.isHidden = true
    }
}

// Удалить все subviews (старый, но всё ещё встречающийся способ)
contentView.subviews.forEach { $0.removeFromSuperview() }
```

#### 2. Работа с порядком слоёв (z-order)

```swift
// Поднять кнопку на передний план
contentView.bringSubviewToFront(loginButton)

// Опустить фон под всё остальное
contentView.sendSubviewToBack(backgroundImageView)

// Поменять местами два слоя
contentView.exchangeSubview(at: 0, withSubviewAt: 1)
```

#### 3. Проверка наличия определённой вью среди subviews

```swift
if contentView.subviews.contains(where: { $0 is LoadingIndicatorView }) {
    // уже есть индикатор загрузки
} else {
    let loader = LoadingIndicatorView()
    contentView.addSubview(loader)
}
```

#### 4. Рекурсивный обход всей иерархии (самый мощный паттерн)

```swift
extension UIView {
    func findAllSubviews<T: UIView>(ofType: T.Type) -> [T] {
        var result: [T] = []
        
        if let found = self as? T {
            result.append(found)
        }
        
        for subview in subviews {
            result.append(contentsOf: subview.findAllSubviews(ofType: T.self))
        }
        
        return result
    }
}

// Пример использования
let allButtons = view.findAllSubviews(ofType: UIButton.self)
allButtons.forEach { $0.isEnabled = false }
```

#### 5. Работа с subviews в [[layoutSubviews]] (очень частый кейс)

```swift
override func layoutSubviews() {
    super.layoutSubviews()
    
    // Размещаем все subviews вручную
    for (index, subview) in subviews.enumerated() {
        let x = CGFloat(index) * 80
        subview.frame = CGRect(x: x, y: 0, width: 70, height: bounds.height)
    }
}
```

### Лучшие практики работы с subviews в Swift 2026

- **Не храни сильные ссылки на subviews в свойствах**, если они добавляются через addSubview — это создаёт retain cycle  
  Правильно: используй IBOutlet или lazy var + addSubview в init

```swift
@IBOutlet weak var titleLabel: UILabel!  // правильно (weak)
lazy var badgeView: UIView = {            // правильно (добавляем в addSubview)
    let v = UIView()
    addSubview(v)
    return v
}()
```

- **Используй subviews.reversed()** при поиске сверху вниз ([[hitTest]], drawing, gesture priority)
- **Очищай subviews** через `for view in subviews { view.removeFromSuperview() }` — безопасно
- **Не модифицируй subviews внутри перебора subviews** — это может привести к крашу (concurrent modification)

```swift
// Плохо — может крашнуться
for subview in subviews {
    subview.removeFromSuperview()  // модификация во время итерации
}

// Хорошо
subviews.forEach { $0.removeFromSuperview() }
```

- **Swift 6 strict concurrency** — все операции с subviews должны быть на @MainActor  
- **Документируйте** — пиши комментарий «subviews — все прямые дочерние вью для ручного layout»

**Короткий девиз 2026**:
> subviews — это **живой массив прямых детей** твоей вью.  
> Он определяет z-порядок (порядок отрисовки), используется для перебора, очистки, ручного layout и отладки.  
> Никогда не храни их как [[strong]] свойства (кроме [[@IBOutlet]] [[weak]]), всегда итерируй безопасно и помни: порядок в массиве = порядок слоёв.
