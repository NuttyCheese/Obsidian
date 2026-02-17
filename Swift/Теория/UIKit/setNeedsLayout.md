**`setNeedsLayout()`** — это метод класса `UIView` в UIKit, который **откладывает** перерасчёт макета (layout) до следующего подходящего момента в цикле обновления интерфейса.

Он **не заставляет** систему пересчитывать макет прямо сейчас (в отличие от `layoutIfNeeded()`), а просто **помечает** представление как «грязное» — «мне нужно обновить расположение subviews».

Это один из самых важных методов для **оптимизации** производительности в UIKit.

### Когда и зачем вызывать `setNeedsLayout()`

| Ситуация / изменение                                   | Нужно ли вызывать `setNeedsLayout()`? | Почему именно так |
|--------------------------------------------------------|---------------------------------------|-------------------|
| Изменил `constant` существующего констрейнта           | **Да**                                | Auto Layout не узнает об изменении без метки |
| Добавил/удалил subview программно                      | **Да** (часто)                        | Система не всегда автоматически помечает родителя |
| Изменил свойство, влияющее на intrinsicContentSize     | **Да**                                | Например, текст в UILabel стал длиннее |
| Изменил `hidden`, `alpha` или `isUserInteractionEnabled` subview | **Обычно нет**                        | Эти свойства не влияют на layout |
| Вызвал `layoutIfNeeded()` на ребёнке                   | **Нет**                               | Родитель уже помечен как нуждающийся |
| Внутри `layoutSubviews()`                              | **Никогда**                           | Иначе бесконечный цикл |
| Хочу немедленный пересчёт (сразу знать новые bounds)   | **Нет** → используй `layoutIfNeeded()` | `setNeedsLayout()` откладывает |

### Как работает `setNeedsLayout()` под капотом

1. Вызов `setNeedsLayout()`  
   → флаг `needsLayout` у вью становится `true`

2. Система в конце текущего run loop (или при следующей перерисовке) видит флаг  
   → вызывает `layoutSubviews()` на этой вью и всех её предках, у которых тоже стоит флаг

3. После вызова `layoutSubviews()` флаг сбрасывается

**Важно**:  
Один вызов `setNeedsLayout()` на вложенной вью **автоматически** помечает всех родителей до корня.  
Поэтому достаточно вызвать на самой глубокой изменённой вью.

### Самый популярный и правильный паттерн 2026 года

```swift
final class ProfileHeaderView: UIView {
    
    private let nameLabel = UILabel()
    private var nameTopConstraint: NSLayoutConstraint!
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupSubviews()
        setupConstraints()
    }
    
    private func setupSubviews() {
        nameLabel.translatesAutoresizingMaskIntoConstraints = false
        addSubview(nameLabel)
        nameLabel.font = .preferredFont(forTextStyle: .headline)
    }
    
    private func setupConstraints() {
        nameTopConstraint = nameLabel.topAnchor.constraint(equalTo: safeAreaLayoutGuide.topAnchor, constant: 16)
        
        NSLayoutConstraint.activate([
            nameTopConstraint,
            nameLabel.centerXAnchor.constraint(equalTo: centerXAnchor)
        ])
    }
    
    // Метод, который вызывается, когда нужно сдвинуть label ниже
    func showExtraContent() {
        nameTopConstraint.constant = 80  // сдвигаем вниз
        
        // Самое важное — помечаем, что нужно пересчитать layout
        setNeedsLayout()
        
        // НЕ вызываем layoutIfNeeded() здесь, если не нужны размеры СЕЙЧАС
    }
    
    // Если размеры нужны немедленно (например, для расчёта высоты)
    func showExtraContentAndMeasure() {
        nameTopConstraint.constant = 80
        setNeedsLayout()
        
        // Теперь layout пересчитан
        layoutIfNeeded()
        
        print("Новая высота заголовка: \(bounds.height)")
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

### Когда использовать именно `setNeedsLayout()` (а не `layoutIfNeeded()`)

| Сценарий                                      | Правильный выбор                  | Почему |
|-----------------------------------------------|-----------------------------------|-------|
| Изменил констрейнт в ответ на пользовательский ввод | `setNeedsLayout()`                | Обновление можно отложить до следующего кадра — плавнее UI |
| Нужно знать новые размеры **прямо сейчас** (для расчёта, анимации) | `layoutIfNeeded()`                | Немедленный пересчёт |
| В цикле (например, скролл, анимация)          | `setNeedsLayout()`                | Не тормозит текущий кадр |
| В `layoutSubviews()` или `draw(_:)`           | **Никогда**                       | Уже идёт пересчёт |
| Динамическая высота ячейки таблицы            | `layoutIfNeeded()` перед измерением | `systemLayoutSizeFitting` требует актуальный layout |

### Самые частые ошибки и как их избежать

| Ошибка                                           | Последствия                                  | Как исправить |
|--------------------------------------------------|----------------------------------------------|---------------|
| Вызов `layoutIfNeeded()` после каждого изменения | Лишние перерасчёты → тормоза                 | Вызывай один раз в конце блока изменений |
| Не вызвал `setNeedsLayout()` после изменения constant | Макет не обновится до следующего естественного layout pass | Всегда вызывай после изменения |
| Вызов `setNeedsLayout()` внутри `layoutSubviews()` | Бесконечный цикл                             | Никогда не делай |
| Ожидание немедленного обновления после `setNeedsLayout()` | `bounds` остаются старыми                    | Вызывай `layoutIfNeeded()` если нужны актуальные размеры |

### Короткий девиз 2026

> `setNeedsLayout()` — это «скажи системе: у меня изменился макет, сделай перерасчёт **позже**, когда будет удобно».  
> `layoutIfNeeded()` — это «сделай перерасчёт **прямо сейчас**, мне нужны актуальные размеры немедленно».  
> `layoutSubviews()` — это «место, где я сам расставляю subviews вручную» (только переопределять, не вызывать).

Удачи с эффективным и безлаговым Auto Layout в твоём приложении! 📏