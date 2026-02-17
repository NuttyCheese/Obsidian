**`layoutIfNeeded()`** — это метод класса [[UIView]], который **немедленно** заставляет систему пересчитать и применить текущий [[Auto Layout]] для этого представления и всех его подпредставлений.

Он нужен в ситуациях, когда вы **не можете ждать** следующего кадра рендеринга (next [[RunLoop]]), а вам **прямо сейчас** нужны актуальные размеры/положения после изменения constraints или [[frame]].

### Когда `layoutIfNeeded()` действительно необходим (2026 реальные кейсы)

| Ситуация                                      | Почему без `layoutIfNeeded()` сломается              | Где именно вызывать |
|-----------------------------------------------|-------------------------------------------------------|---------------------|
| Изменение constraints → сразу нужен новый размер | `view.bounds` или `subview.frame` всё ещё старые      | После `activate` или изменения constant |
| Программное создание вью → нужно измерить до показа | `systemLayoutSizeFitting` вернёт некорректный размер   | После `addSubview` + constraints |
| Анимация, зависящая от финального размера      | `frame` берётся до layout pass                        | Внутри `UIView.animate` или `layoutSubviews` |
| Кастомный layout (в `layoutSubviews`)          | Нужно знать актуальные размеры детей                  | В начале `layoutSubviews()` |
| Расчёт динамической высоты ячейки таблицы      | `systemLayoutSizeFitting` требует актуальный layout   | В `heightForRowAt` или data source |
| Снятие скриншота / рендер в изображение        | `snapshotView` или `UIGraphicsImageRenderer` берут старый layout | Перед `snapshotView(afterScreenUpdates: true)` |

### Самый частый и правильный паттерн 2026 года

```swift
final class ProfileHeaderView: UIView {
    
    private let avatarImageView = UIImageView()
    private let nameLabel = UILabel()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupSubviews()
        setupConstraints()
    }
    
    private func setupSubviews() {
        [avatarImageView, nameLabel].forEach {
            $0.translatesAutoresizingMaskIntoConstraints = false
            addSubview($0)
        }
    }
    
    private func setupConstraints() {
        NSLayoutConstraint.activate([
            avatarImageView.topAnchor.constraint(equalTo: safeAreaLayoutGuide.topAnchor, constant: 16),
            avatarImageView.centerXAnchor.constraint(equalTo: centerXAnchor),
            avatarImageView.widthAnchor.constraint(equalToConstant: 80),
            avatarImageView.heightAnchor.constraint(equalTo: avatarImageView.widthAnchor),
            
            nameLabel.topAnchor.constraint(equalTo: avatarImageView.bottomAnchor, constant: 12),
            nameLabel.centerXAnchor.constraint(equalTo: centerXAnchor)
        ])
    }
    
    // Самый важный момент — когда нам нужны актуальные размеры ДО показа
    func calculateHeightForWidth(_ width: CGFloat) -> CGFloat {
        // 1. Устанавливаем временную ширину
        frame.size.width = width
        
        // 2. Немедленно применяем Auto Layout
        layoutIfNeeded()
        
        // 3. Теперь можно безопасно измерять
        let height = systemLayoutSizeFitting(
            CGSize(width: width, height: UIView.layoutFittingCompressedSize.height),
            withHorizontalFittingPriority: .required,
            verticalFittingPriority: .fittingSizeLevel
        ).height
        
        return height
    }
}
```

### Самые опасные ловушки и правильные места вызова

| Где **НЕЛЬЗЯ** вызывать `layoutIfNeeded()` | Почему сломается / тормозит                                               | Что делать вместо                                          |
| ------------------------------------------ | ------------------------------------------------------------------------- | ---------------------------------------------------------- |
| Внутри `layoutSubviews()`                  | Бесконечный цикл (layout → layoutIfNeeded → layout)                       | Вызывать **до** [[layoutSubviews]]                         |
| В `draw(_:)`                               | Тормозит рендеринг, бессмысленно                                          | Вызывать **до** draw                                       |
| В `viewDidLoad()` без необходимости        | Часто бесполезно — constraints ещё не применены                           | Вызывать после `addSubview` + `activate`                   |
| В цикле анимации                           | Очень дорого — лучше использовать `layoutIfNeeded()` один раз до анимации | Использовать `UIView.animate` + `layoutIfNeeded` до старта |

### Правильные места вызова (2026 золотой стандарт)

1. **После изменения constraints** → `layoutIfNeeded()` → получаем актуальные `bounds`  
2. **Перед вызовом `systemLayoutSizeFitting`** → для динамической высоты ячеек  
3. **В `layoutSubviews()`** — **никогда** не вызывать `layoutIfNeeded()` на себя, только на детей если нужно  
4. **Перед snapshot / render в изображение** — `layoutIfNeeded()` → `snapshotView(afterScreenUpdates: true)`  
5. **В `viewDidLayoutSubviews()`** — уже поздно, там layout уже выполнен

### Короткий девиз 2026

> `layoutIfNeeded()` — это «сделай layout **прямо сейчас**, не жди следующего кадра».  
> Вызывай его **только** когда тебе **немедленно** нужны актуальные `bounds`, `frame`, `systemLayoutSizeFitting` или перед рендером.  
> В 2026 году это **единственный** способ принудительно обновить Auto Layout вне естественного цикла.
