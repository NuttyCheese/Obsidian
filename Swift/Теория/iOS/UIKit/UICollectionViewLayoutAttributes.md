#uikit #uicollectionview #layout #animation #ios #swift

---

## UICollectionViewLayoutAttributes — Атрибуты макета для ячеек

### Определение

**`UICollectionViewLayoutAttributes`** — это класс в [[UIKit]], который инкапсулирует **все параметры отображения** элемента (ячейки, supplementary view или decoration view) в `UICollectionView`. Он содержит такие свойства, как позиция (`frame`), размер, прозрачность (`alpha`), трансформация (`transform`), z-индекс (`zIndex`) и т.д.

В кастомных `UICollectionViewLayout` вы создаёте и возвращаете экземпляры этого класса для каждого элемента, определяя его положение и внешний вид. `UICollectionView` затем использует эти атрибуты для рендеринга элементов.

### Зачем это знать iOS-разработчику?

1.  **Кастомные макеты:** Создание нестандартных расположений ячеек (круговые, зигзагообразные, Pinterest-подобные, карусели).
2.  **Анимация:** Изменение атрибутов при скролле (эффекты параллакса, масштабирование, вращение).
3.  **Динамические макеты:** Адаптация под размер экрана, ориентацию, динамические данные.
4.  **Supplementary views:** Управление хедерами, футерами и декоративными элементами.
5.  **Производительность:** Кэширование атрибутов для быстрого доступа.

---

### Ключевые свойства

| Свойство                         | Тип                          | Описание                                                        |
| -------------------------------- | ---------------------------- | --------------------------------------------------------------- |
| **`frame`**                      | [[CGRect]]                   | Прямоугольник элемента в координатах [[UICollectionView]].      |
| **`center`**                     | [[CGPoint]]                  | Центр элемента (альтернатива `frame`).                          |
| **`size`**                       | [[CGSize]]                   | Размер элемента (ширина, высота).                               |
| **`transform3D`**                | [[CATransform3D]]            | 3D-трансформация элемента.                                      |
| **`transform`**                  | [[CGAffineTransform]]        | 2D-трансформация элемента.                                      |
| **`alpha`**                      | [[CGFloat]]                  | Прозрачность (0.0 — полностью прозрачный, 1.0 — непрозрачный).  |
| **`zIndex`**                     | [[Int]]                      | Порядок наложения (чем выше, тем выше).                         |
| **`isHidden`**                   | [[Bool]]                     | Скрыт ли элемент.                                               |
| **`representedElementCategory`** | `RepresentedElementCategory` | Тип элемента: `.cell`, `.supplementaryView`, `.decorationView`. |
| **`indexPath`**                  | [[IndexPath]]                | Путь к элементу (секция, элемент).                              |

---

### Создание атрибутов

```swift
import UIKit

class CustomLayout: UICollectionViewLayout {
    override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
        var attributesArray: [UICollectionViewLayoutAttributes] = []
        
        // Для каждого элемента в секции
        for section in 0..<collectionView?.numberOfSections ?? 0 {
            for item in 0..<(collectionView?.numberOfItems(inSection: section) ?? 0) {
                let indexPath = IndexPath(item: item, section: section)
                
                // Создаём атрибуты для ячейки
                let attributes = UICollectionViewLayoutAttributes(forCellWith: indexPath)
                attributes.frame = CGRect(x: 10, y: CGFloat(item) * 100, width: 200, height: 80)
                attributesArray.append(attributes)
            }
        }
        
        return attributesArray
    }
}
```

---

### Типы атрибутов

| Метод создания | Назначение |
|---|---|
| **`forCell(with:)`** | Для обычных ячеек (`UICollectionViewCell`). |
| **`forSupplementaryView(ofKind:with:)`** | Для supplementary views (хедеры, футеры). |
| **`forDecorationView(ofKind:with:)`** | Для декоративных элементов (фоны, разделители). |

```swift
// Supplementary view (хедер)
let headerAttributes = UICollectionViewLayoutAttributes(
    forSupplementaryViewOfKind: UICollectionView.elementKindSectionHeader,
    with: indexPath
)

// Decoration view
let decorationAttributes = UICollectionViewLayoutAttributes(
    forDecorationViewOfKind: "CustomDecoration",
    with: indexPath
)
```

---

### Пример 1: Простой кастомный макет

```swift
class SimpleCustomLayout: UICollectionViewLayout {
    private var attributesCache: [IndexPath: UICollectionViewLayoutAttributes] = [:]
    private var contentHeight: CGFloat = 0
    
    override func prepare() {
        super.prepare()
        attributesCache.removeAll()
        contentHeight = 0
        
        guard let collectionView = collectionView else { return }
        
        let numberOfSections = collectionView.numberOfSections
        var yOffset: CGFloat = 0
        
        for section in 0..<numberOfSections {
            let numberOfItems = collectionView.numberOfItems(inSection: section)
            
            for item in 0..<numberOfItems {
                let indexPath = IndexPath(item: item, section: section)
                let attributes = UICollectionViewLayoutAttributes(forCellWith: indexPath)
                
                // Располагаем ячейки в колонку
                let width = collectionView.bounds.width - 20
                let height: CGFloat = 80
                let x: CGFloat = 10
                let y = yOffset
                
                attributes.frame = CGRect(x: x, y: y, width: width, height: height)
                attributesCache[indexPath] = attributes
                
                yOffset += height + 10
            }
        }
        
        contentHeight = yOffset
    }
    
    override var collectionViewContentSize: CGSize {
        return CGSize(width: collectionView?.bounds.width ?? 0, height: contentHeight)
    }
    
    override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
        return attributesCache.values.filter { rect.intersects($0.frame) }
    }
    
    override func layoutAttributesForItem(at indexPath: IndexPath) -> UICollectionViewLayoutAttributes? {
        return attributesCache[indexPath]
    }
}
```

---

### Пример 2: Pinterest-подобный макет

```swift
class PinterestLayout: UICollectionViewLayout {
    private var attributesCache: [IndexPath: UICollectionViewLayoutAttributes] = [:]
    private var columnHeights: [CGFloat] = []
    private var contentHeight: CGFloat = 0
    
    let numberOfColumns = 2
    let cellPadding: CGFloat = 8
    
    override func prepare() {
        super.prepare()
        attributesCache.removeAll()
        contentHeight = 0
        
        guard let collectionView = collectionView else { return }
        
        let columnWidth = (collectionView.bounds.width - cellPadding * CGFloat(numberOfColumns + 1)) / CGFloat(numberOfColumns)
        columnHeights = Array(repeating: 0, count: numberOfColumns)
        
        let numberOfSections = collectionView.numberOfSections
        
        for section in 0..<numberOfSections {
            let numberOfItems = collectionView.numberOfItems(inSection: section)
            
            for item in 0..<numberOfItems {
                let indexPath = IndexPath(item: item, section: section)
                let attributes = UICollectionViewLayoutAttributes(forCellWith: indexPath)
                
                // Выбираем колонку с наименьшей высотой
                let column = columnHeights.enumerated().min(by: { $0.element < $1.element })?.offset ?? 0
                let x = cellPadding + CGFloat(column) * (columnWidth + cellPadding)
                let y = columnHeights[column] + cellPadding
                
                // Имитация разной высоты ячеек (в реальности — из данных)
                let randomHeight: CGFloat = CGFloat(arc4random_uniform(100) + 80)
                let frame = CGRect(x: x, y: y, width: columnWidth, height: randomHeight)
                
                attributes.frame = frame
                attributesCache[indexPath] = attributes
                
                columnHeights[column] = frame.maxY
                contentHeight = max(contentHeight, frame.maxY)
            }
        }
    }
    
    override var collectionViewContentSize: CGSize {
        return CGSize(width: collectionView?.bounds.width ?? 0, height: contentHeight + cellPadding)
    }
    
    override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
        return attributesCache.values.filter { rect.intersects($0.frame) }
    }
    
    override func layoutAttributesForItem(at indexPath: IndexPath) -> UICollectionViewLayoutAttributes? {
        return attributesCache[indexPath]
    }
}
```

---

### Пример 3: Эффект параллакса при скролле

```swift
class ParallaxLayout: UICollectionViewFlowLayout {
    override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
        guard let attributes = super.layoutAttributesForElements(in: rect) else { return nil }
        guard let collectionView = collectionView else { return attributes }
        
        let offsetY = collectionView.contentOffset.y
        let delta = offsetY > 0 ? offsetY : 0
        
        for attribute in attributes {
            if attribute.representedElementCategory == .cell {
                // Параллакс: ячейки смещаются в зависимости от скролла
                let originalFrame = attribute.frame
                let newY = originalFrame.origin.y + delta * 0.2
                attribute.frame = CGRect(x: originalFrame.origin.x, y: newY,
                                        width: originalFrame.width, height: originalFrame.height)
                
                // Масштабирование при скролле
                let distanceFromCenter = abs(attribute.center.y - collectionView.bounds.midY)
                let scale = 1.0 - min(distanceFromCenter / 500, 0.3)
                attribute.transform = CGAffineTransform(scaleX: scale, y: scale)
            }
        }
        
        return attributes
    }
    
    override func shouldInvalidateLayout(forBoundsChange newBounds: CGRect) -> Bool {
        return true
    }
}
```

---

### Анимация атрибутов

```swift
class AnimatedLayout: UICollectionViewFlowLayout {
    override func initialLayoutAttributesForAppearingItem(at itemIndexPath: IndexPath) -> UICollectionViewLayoutAttributes? {
        let attributes = super.initialLayoutAttributesForAppearingItem(at: itemIndexPath)
        
        // Начальная анимация: ячейка появляется с масштабированием
        attributes?.transform = CGAffineTransform(scaleX: 0.1, y: 0.1)
        attributes?.alpha = 0
        
        return attributes
    }
    
    override func finalLayoutAttributesForDisappearingItem(at itemIndexPath: IndexPath) -> UICollectionViewLayoutAttributes? {
        let attributes = super.finalLayoutAttributesForDisappearingItem(at: itemIndexPath)
        
        // Конечная анимация: ячейка исчезает с поворотом
        attributes?.transform = CGAffineTransform(rotationAngle: .pi)
        attributes?.alpha = 0
        
        return attributes
    }
}
```

---

### Работа с supplementary views

```swift
class HeaderFooterLayout: UICollectionViewFlowLayout {
    override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
        guard let attributes = super.layoutAttributesForElements(in: rect) else { return nil }
        
        var mutableAttributes = attributes
        
        for attribute in mutableAttributes {
            if attribute.representedElementKind == UICollectionView.elementKindSectionHeader {
                // Фиксируем хедер при скролле
                guard let collectionView = collectionView else { continue }
                let offsetY = collectionView.contentOffset.y
                let headerFrame = attribute.frame
                
                if offsetY > headerFrame.origin.y && offsetY < headerFrame.origin.y + headerFrame.height {
                    let newY = max(offsetY, headerFrame.origin.y)
                    attribute.frame = CGRect(x: headerFrame.origin.x, y: newY,
                                            width: headerFrame.width, height: headerFrame.height)
                    attribute.zIndex = 10
                }
            }
        }
        
        return mutableAttributes
    }
    
    override func shouldInvalidateLayout(forBoundsChange newBounds: CGRect) -> Bool {
        return true
    }
}
```

---

### Кэширование атрибутов для производительности

```swift
class CachedLayout: UICollectionViewLayout {
    private var cache: [IndexPath: UICollectionViewLayoutAttributes] = [:]
    private var contentSize: CGSize = .zero
    
    override func prepare() {
        super.prepare()
        cache.removeAll()
        
        guard let collectionView = collectionView else { return }
        
        // Генерируем и кэшируем атрибуты для всех элементов
        let numberOfSections = collectionView.numberOfSections
        var yOffset: CGFloat = 0
        
        for section in 0..<numberOfSections {
            let numberOfItems = collectionView.numberOfItems(inSection: section)
            var maxHeight: CGFloat = 0
            
            for item in 0..<numberOfItems {
                let indexPath = IndexPath(item: item, section: section)
                let attributes = UICollectionViewLayoutAttributes(forCellWith: indexPath)
                
                // Вычисляем frame
                let width = collectionView.bounds.width
                let height: CGFloat = 100
                let frame = CGRect(x: 0, y: yOffset, width: width, height: height)
                
                attributes.frame = frame
                cache[indexPath] = attributes
                
                maxHeight = max(maxHeight, frame.maxY)
            }
            
            yOffset += maxHeight
        }
        
        contentSize = CGSize(width: collectionView.bounds.width, height: yOffset)
    }
    
    override var collectionViewContentSize: CGSize {
        return contentSize
    }
    
    override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
        return cache.values.filter { rect.intersects($0.frame) }
    }
    
    override func layoutAttributesForItem(at indexPath: IndexPath) -> UICollectionViewLayoutAttributes? {
        return cache[indexPath]
    }
}
```

---

### Лучшие практики

1.  **Всегда переопределяйте `prepare()`** для предварительного расчёта атрибутов.
2.  **Кэшируйте атрибуты**, чтобы не пересчитывать их каждый раз.
3.  **Используйте `shouldInvalidateLayout(forBoundsChange:)`** для обновления при скролле (например, для эффектов параллакса).
4.  **Для анимации используйте `initialLayoutAttributesForAppearingItem` и `finalLayoutAttributesForDisappearingItem`.**
5.  **Не вычисляйте атрибуты в `layoutAttributesForElements` повторно** — используйте кэш.

```swift
// Правильно
override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
    return cache.values.filter { rect.intersects($0.frame) }
}

// Неправильно (ресурсоёмко)
override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
    var attributes: [UICollectionViewLayoutAttributes] = []
    for section in 0..<sections {
        for item in 0..<items {
            // вычисление frame
        }
    }
    return attributes
}
```

---

### Итог

**`UICollectionViewLayoutAttributes`** — это центральный элемент кастомных макетов `UICollectionView`. Он позволяет:

1.  **Определять позицию, размер, прозрачность, трансформацию** каждого элемента.
2.  **Создавать динамические макеты** (Pinterest, карусели, круговые).
3.  **Анимировать появление и исчезновение** элементов.
4.  **Добавлять эффекты при скролле** (параллакс, масштабирование, вращение).
5.  **Управлять хедерами, футерами и декоративными элементами.**

Понимание `UICollectionViewLayoutAttributes` необходимо для создания гибких, анимированных и нестандартных макетов коллекций .