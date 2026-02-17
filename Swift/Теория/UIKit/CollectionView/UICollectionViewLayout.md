**UICollectionViewLayout** — это **абстрактный базовый класс** в UIKit, который определяет, **как именно** должны располагаться ячейки, supplementary views (заголовки/футеры) и decoration views (декоративные элементы) внутри `UICollectionView`.

Это **основа** всей системы макетов коллекции.  
Все конкретные макеты (FlowLayout, Compositional Layout и любые кастомные) наследуются от него.

### Основные подклассы и их использование в 2026 году

| Макет                                   | Когда использовать в 2026 году                          | Сложность | Рекомендация |
|-----------------------------------------|----------------------------------------------------------|-----------|--------------|
| **UICollectionViewFlowLayout**          | Простая сетка, горизонтальная/вертикальная карусель, равномерные отступы | Низкая    | Для большинства случаев (если не нужен супер-сложный дизайн) |
| **UICollectionViewCompositionalLayout** | Сложные, современные макеты: разные размеры ячеек, группы, nested groups, orthogonal scrolling | Средняя   | **Основной выбор** для новых проектов 2023–2026 |
| **Кастомный UICollectionViewLayout**    | Полностью уникальный макет (Pinterest, waterfall, circle, helix, custom physics и т.д.) | Высокая   | Только если Compositional Layout не хватает |

### Что нужно реализовать в кастомном UICollectionViewLayout

Минимальный набор методов, которые Apple требует переопределить:

| Метод                                      | Что возвращает / делает                                  | Обязательный? | Самый частый пример |
|--------------------------------------------|----------------------------------------------------------|---------------|---------------------|
| `prepare()`                                | Подготовка макета (расчёты один раз перед показом)       | Да            | Вычислить все атрибуты заранее |
| `layoutAttributesForElements(in:)`         | Атрибуты всех видимых элементов в заданном rect          | Да            | Возвращать массив атрибутов |
| `layoutAttributesForItem(at:)`             | Атрибуты одной конкретной ячейки по indexPath            | Да            | Позиция, размер, zIndex |
| `collectionViewContentSize`                | Общий размер контента (для скролла)                      | Да            | CGSize(width: ..., height: ...) |
| `shouldInvalidateLayout(forBoundsChange:)` | Нужно ли пересчитывать layout при изменении bounds       | Опционально   | true для динамического размера |
| `layoutAttributesForSupplementaryView(...)` | Атрибуты заголовков/футеров                            | Опционально   | Для sticky headers |
| `layoutAttributesForDecorationView(...)`   | Атрибуты декоративных элементов                          | Опционально   | Фон секции, разделители |

### Самый простой кастомный пример (2026 стиль)

```swift
final class WaterfallLayout: UICollectionViewLayout {
    
    private var attributesCache: [UICollectionViewLayoutAttributes] = []
    private var contentHeight: CGFloat = 0
    
    override func prepare() {
        super.prepare()
        
        guard let collectionView = collectionView else { return }
        
        attributesCache.removeAll()
        contentHeight = 0
        
        let numberOfItems = collectionView.numberOfItems(inSection: 0)
        for item in 0..<numberOfItems {
            let indexPath = IndexPath(item: item, section: 0)
            let attributes = UICollectionViewLayoutAttributes(forCellWith: indexPath)
            
            // Пример: случайная высота
            let height = CGFloat.random(in: 100...300)
            let width = collectionView.bounds.width / 2 - 10
            let x = item % 2 == 0 ? 5 : width + 15
            let y = contentHeight
            
            attributes.frame = CGRect(x: x, y: y, width: width, height: height)
            attributesCache.append(attributes)
            
            contentHeight += height + 10  // отступ между строками
        }
    }
    
    override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
        return attributesCache.filter { $0.frame.intersects(rect) }
    }
    
    override func layoutAttributesForItem(at indexPath: IndexPath) -> UICollectionViewLayoutAttributes? {
        return attributesCache.first { $0.indexPath == indexPath }
    }
    
    override var collectionViewContentSize: CGSize {
        return CGSize(width: collectionView?.bounds.width ?? 0, height: contentHeight)
    }
}
```

### Лучшие практики UICollectionViewLayout в 2026

- **Предпочитай Compositional Layout** — в 90 % случаев он покрывает все нужды  
- **Кастомный layout** — только если Compositional не хватает (очень сложные анимации, physics, waterfall без фиксированных колонок)  
- **prepare()** — делай все тяжёлые расчёты здесь (один раз)  
- **layoutAttributesForElements(in:)** — фильтруй по rect, иначе будет тормозить  
- **shouldInvalidateLayout** — возвращай true, если layout зависит от bounds  
- **@MainActor** — весь layout-класс — на главном акторе  
- **Swift 6 strict concurrency** — UICollectionViewLayout полностью безопасен  
- **Документируйте** — пиши комментарий «UICollectionViewLayout — кастомный waterfall layout с динамической высотой»

**Короткий девиз 2026**:
> UICollectionViewLayout — это когда ты хочешь **полностью контролировать**, где и как лежат ячейки, заголовки и декорации в коллекции.  
> В 2026 году **первый выбор** — Compositional Layout (встроенный).  
> Кастомный подкласс — только для очень сложных или уникальных макетов.

Удачи с красивой и производительной коллекцией! 📱