**`layoutIfNeeded()`**, **`setNeedsLayout()`** и **`layoutSubviews()`** — это три разных инструмента в UIKit для управления Auto Layout и размещением подвью. Они решают похожие задачи, но **работают на разных уровнях** и вызываются в разное время.

Вот максимально чёткое и практичное сравнение (актуально на 2026 год):

| Метод                  | Что делает (простыми словами)                              | Когда вызывается системой | Можно ли вызвать вручную | Вызывает `layoutSubviews()`? | Самый частый сценарий использования в 2026 | Когда **НЕ** использовать |
|------------------------|-------------------------------------------------------------|----------------------------|---------------------------|--------------------------------|--------------------------------------------|----------------------------|
| **`setNeedsLayout()`** | «Эй, система, у меня изменились constraints — запланируй перерасчёт макета на следующий кадр» | Никогда не вызывается автоматически | Да                        | Нет (только планирует)         | После изменения constant, добавления/удаления constraints, если не нужны размеры сразу | Когда нужны актуальные `bounds` / `frame` **прямо сейчас** |
| **`layoutIfNeeded()`** | «Сделай перерасчёт макета **прямо сейчас**, не жди следующего кадра» | Никогда не вызывается автоматически | Да                        | **Да** (немедленно)            | Когда нужны актуальные `bounds`, `frame`, `systemLayoutSizeFitting` **до** следующего кадра | Внутри `layoutSubviews()` — вызовет бесконечный цикл |
| **`layoutSubviews()`** | Метод жизненного цикла — здесь **реально** размещаются subviews | Когда система решила пересчитать layout (после `setNeedsLayout`, изменения размеров и т.д.) | Нет (только переопределяют) | — (это и есть он сам)          | Переопределение для кастомного размещения subviews (например, вручную расставить frame) | Вызывать вручную — это почти всегда ошибка |

### Короткая шпаргалка по выбору (2026)

| Ситуация / вопрос                                      | Что использовать                     | Почему именно это |
|--------------------------------------------------------|--------------------------------------|-------------------|
| Изменил constraint → хочу **сразу** знать новый размер | `layoutIfNeeded()`                   | Немедленный пересчёт → `bounds` актуальны |
| Изменил constraint → обновление можно отложить до кадра | `setNeedsLayout()`                   | Оптимизация — система сама решит, когда пересчитать |
| Нужно **кастомно** разместить subviews (не только Auto Layout) | Переопределить `layoutSubviews()`    | Здесь ты сам расставляешь frame, center и т.д. |
| Делаю динамическую высоту ячейки таблицы               | `layoutIfNeeded()` перед `systemLayoutSizeFitting` | Без него размер будет старый |
| Внутри `layoutSubviews()` нужно пересчитать детей      | **Никогда** не вызывай `layoutIfNeeded()` на себя | Бесконечный цикл. Вызывай на детях, если нужно |
| Анимация зависит от финального размера                 | `layoutIfNeeded()` **до** начала анимации | Чтобы знать, откуда и куда анимировать |

### Самый частый и правильный пример 2026 года

```swift
final class DynamicHeightCell: UITableViewCell {
    
    private let label = UILabel()
    
    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        label.numberOfLines = 0
        contentView.addSubview(label)
        label.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            label.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 16),
            label.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 16),
            label.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -16),
            label.bottomAnchor.constraint(equalTo: contentView.bottomAnchor, constant: -16)
        ])
    }
    
    func configure(with text: String) {
        label.text = text
        
        // Важно: без этого systemLayoutSizeFitting вернёт старый размер!
        contentView.layoutIfNeeded()
    }
    
    override func systemLayoutSizeFitting(_ targetSize: CGSize, 
                                         withHorizontalFittingPriority horizontalFittingPriority: UILayoutPriority, 
                                         verticalFittingPriority: UILayoutPriority) -> CGSize {
        return contentView.systemLayoutSizeFitting(targetSize,
                                                  withHorizontalFittingPriority: horizontalFittingPriority,
                                                  verticalFittingPriority: verticalFittingPriority)
    }
}
```

### Самые опасные ошибки и как их избежать

| Ошибка                                           | Последствия                                  | Как исправить |
|--------------------------------------------------|----------------------------------------------|---------------|
| Вызов `layoutIfNeeded()` внутри `layoutSubviews()` | Бесконечный цикл → приложение зависает      | Вызывать только **до** или **вне** `layoutSubviews` |
| Изменение constraints → чтение `bounds` без `layoutIfNeeded()` | Получаешь старые размеры                     | Всегда `layoutIfNeeded()` перед чтением |
| `setNeedsLayout()` вместо `layoutIfNeeded()` когда нужны размеры сразу | Размеры не обновятся до следующего кадра     | Используй `layoutIfNeeded()` для немедленного эффекта |
| Забыл вызвать `super.layoutSubviews()`           | Сломанная иерархия, subviews не размещаются  | Всегда вызывай super первым |

**Короткий девиз 2026**:
> - `setNeedsLayout()` — «запланируй перерасчёт на потом» (эффективно)  
> - `layoutIfNeeded()` — «сделай перерасчёт **прямо сейчас**» (когда нужны актуальные размеры немедленно)  
> - `layoutSubviews()` — «я переопределяю **как именно** размещать subviews» (только переопределять, не вызывать вручную)

Удачи с точным и безлаговым Auto Layout в Swift! 📏