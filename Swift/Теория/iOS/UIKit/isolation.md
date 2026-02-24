**Isolation** — это один из самых важных и одновременно самых недооценённых паттернов в UIKit-разработке 2025–2026 годов.

Он решает проблему **"кто должен обрабатывать касание / событие / жест"**, когда в иерархии представлений несколько слоёв могут претендовать на ввод (например, прозрачный оверлей над кнопкой, плавающая панель над таблицей, кастомный контейнер и т.д.).

### Что такое isolation в контексте UIKit (2026)

Isolation — это техника, при которой вью **изолирует** (отрезает) события от своих subviews или, наоборот, **передаёт** их дальше, **игнорируя** собственную геометрию.

Главные инструменты для реализации isolation:

| Метод / Свойство                     | Что делает                                                                 | Самый частый сценарий использования в 2026 |
|--------------------------------------|----------------------------------------------------------------------------|---------------------------------------------|
| `hitTest(_:with:)`                   | Переопределяется, чтобы вернуть другую вью или nil                         | Пропуск касаний сквозь оверлей, расширение зоны кнопки |
| `point(inside:with:)`                | Переопределяется, чтобы изменить логическую область касания               | Увеличение/уменьшение зоны касания без изменения frame |
| `isUserInteractionEnabled = false`   | Полностью отключает обработку касаний (и всех subviews)                    | Временное отключение всей панели |
| `clipsToBounds = false` + `alpha`    | Не влияет на hitTest напрямую, но часто комбинируется                      | — |
| `isHidden = true`                    | Полностью исключает вью и все subviews из hitTest                          | Скрытие слоя без удаления |
| `UIGestureRecognizer` + `cancelsTouchesInView` | Жест может "съесть" событие и не передать touches дальше                   | Перехват свайпов, лонг-прессов |
| `nextResponder`                      | Можно вручную перенаправить touchesBegan/Moved/Ended                       | Редко, но мощно |

### Самые популярные сценарии isolation в 2026 году

#### 1. Пропуск касаний сквозь полупрозрачный оверлей / модальный фон

Самый частый кейс — затемнённый фон модального окна не должен блокировать кнопки под собой.

```swift
final class DimmingView: UIView {
    
    override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
        // Если точка внутри нас — пропускаем (возвращаем nil)
        // Таким образом касание проходит сквозь dimming на нижележащие вью
        let hit = super.hitTest(point, with: event)
        return hit == self ? nil : hit
    }
}
```

#### 2. Расширение зоны касания кнопки / элемента (без изменения визуального frame)

```swift
final class FatTouchButton: UIButton {
    
    private let touchInset: CGFloat = 20
    
    override func point(inside point: CGPoint, with event: UIEvent?) -> Bool {
        // Расширяем зону касания на 20 pt со всех сторон
        let expanded = bounds.insetBy(dx: -touchInset, dy: -touchInset)
        return expanded.contains(point)
    }
}
```

#### 3. Перехват всех касаний определённой областью (floating panel / HUD)

```swift
final class OverlayPanel: UIView {
    
    override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
        // Всегда возвращаем себя, даже если точка не внутри bounds
        // Таким образом панель "ловит" касания за пределами своего frame
        return self
    }
}
```

#### 4. Игнорирование касаний в "дырке" (например, круглый вырез в оверлее)

```swift
final class HoleView: UIView {
    
    private let holeRect = CGRect(x: 100, y: 100, width: 80, height: 80)
    
    override func point(inside point: CGPoint, with event: UIEvent?) -> Bool {
        // Если точка внутри "дырки" — игнорируем (возвращаем false)
        if holeRect.contains(point) {
            return false
        }
        return super.point(inside: point, with: event)
    }
}
```

#### 5. Делегирование касаний в другую вью (кастомный контейнер)

```swift
final class PassthroughContainer: UIView {
    
    weak var targetView: UIView?
    
    override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
        guard let target = targetView else {
            return super.hitTest(point, with: event)
        }
        
        let converted = convert(point, to: target)
        if target.point(inside: converted, with: event) {
            return target
        }
        
        return nil
    }
}
```

### Лучшие практики isolation в UIKit 2026

- **Предпочитайте** переопределять `point(inside:with:)` — это проще и чище, чем полный `hitTest`  
- **Вызывайте** `super.hitTest` / `super.point(inside:)` — иначе сломаете subviews  
- **Никогда** не вызывайте `hitTest` вручную в production-коде — только для дебага  
- **Для SwiftUI** — аналоги: `.allowsHitTesting(false)`, `.contentShape(Rectangle().inset(by: ...))`, `.gesture(..., including: .all)`  
- **Для производительности** — избегайте сложной логики внутри `hitTest` — он вызывается очень часто  
- **Тестируйте** в разных режимах: Split View, Slide Over, iPad с клавиатурой, тёмная тема  
- **Документируйте** — пишите комментарий «hitTest переопределён — пропускаем касания сквозь dimming-оверлей»

**Короткий итог 2026**:
> Isolation в UIKit — это искусство **управления тем, какая вью получит касание** через `hitTest` и `point(inside:)`.  
> В 2026 году:  
> - самый частый кейс — пропуск касаний сквозь оверлей / dimming  
> - второй по популярности — расширение зоны касания кнопки  
> - третий — перехват всех касаний floating-панелью  
> - переопределяйте `point(inside:)` в 80% случаев — это проще и достаточно  
> - вызывайте super — иначе сломаете дочерние вью  
> Это **must-know** для любого UIKit-разработчика, который делает кастомный UI.

Удачи с точным и предсказуемым поведением касаний в твоём приложении! 👆