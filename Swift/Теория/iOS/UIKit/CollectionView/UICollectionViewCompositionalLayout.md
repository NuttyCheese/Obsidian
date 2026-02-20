**`UICollectionViewCompositionalLayout`** — это современный и самый мощный способ создания **композиционных макетов** в `UICollectionView` (доступен с iOS 13, 2019 год).

Это **единственный рекомендуемый** инструмент в 2026 году для любых сложных, адаптивных и красивых коллекций:
- Pinterest-подобные гриды,
- горизонтальные карусели,
- секции с разными размерами ячеек,
- sticky headers / footers,
- orthogonal scrolling (прокрутка секций в разные стороны),
- динамические размеры ячеек,
- группы с разными layouts внутри одной коллекции.

### Почему Compositional Layout — это стандарт 2026 года

| Свойство / Возможность               | UICollectionViewFlowLayout (старый) | UICollectionViewCompositionalLayout (новый) | Выигрыш |
|--------------------------------------|-------------------------------------|---------------------------------------------|---------|
| Гибкость布局                        | Только линейный / вертикальный / горизонтальный | Полностью произвольные секции + группы + items | ★★★★★ |
| Адаптивность к разным размерам экрана | Требует ручных расчётов             | Автоматически адаптируется через fractional / absolute размеры | ★★★★★ |
| Orthogonal scrolling (прокрутка секций вбок) | Нет                                 | Полная поддержка                                | ★★★★★ |
| Nested группы                        | Нет                                 | Группы внутри групп (nested groups)             | ★★★★★ |
| Динамическая высота ячеек            | Сложно (estimatedItemSize)          | Легко через estimated sizes + self-sizing       | ★★★★★ |
| Количество кода                      | Много boilerplate                   | Очень декларативно и мало кода                  | ★★★★★ |

### Основные строительные блоки Compositional Layout

1. **NSCollectionLayoutItem** — один элемент (ячейка)
2. **NSCollectionLayoutGroup** — группа элементов (горизонтальная / вертикальная / кастомная)
3. **NSCollectionLayoutSection** — секция (содержит группы + header/footer + decoration)
4. **NSCollectionLayoutSize** — размер (fractional, absolute, estimated)
5. **NSCollectionLayoutBoundarySupplementaryItem** — header/footer
6. **NSCollectionLayoutDecorationItem** — кастомные фоны / разделители

### Самый популярный и рекомендуемый паттерн 2026 года

#### 1. Простая горизонтальная карусель (самый частый)

```swift
func createCarouselLayout() -> UICollectionViewCompositionalLayout {
    let itemSize = NSCollectionLayoutSize(
        widthDimension: .fractionalWidth(1.0),
        heightDimension: .fractionalHeight(1.0)
    )
    let item = NSCollectionLayoutItem(layoutSize: itemSize)
    
    let groupSize = NSCollectionLayoutSize(
        widthDimension: .fractionalWidth(0.8),   // 80% ширины экрана
        heightDimension: .absolute(200)          // фиксированная высота
    )
    let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitems: [item])
    group.contentInsets = NSDirectionalEdgeInsets(top: 0, leading: 8, bottom: 0, trailing: 8)
    
    let section = NSCollectionLayoutSection(group: group)
    section.orthogonalScrollingBehavior = .groupPagingCentered  // карусель с центрированием
    
    section.interGroupSpacing = 16
    
    return UICollectionViewCompositionalLayout(section: section)
}
```

#### 2. Pinterest-подобный masonry-грид (очень популярно)

```swift
func createMasonryLayout() -> UICollectionViewCompositionalLayout {
    let layout = UICollectionViewCompositionalLayout { sectionIndex, layoutEnvironment in
        
        let itemSize = NSCollectionLayoutSize(
            widthDimension: .fractionalWidth(1.0),
            heightDimension: .estimated(200)  // высота будет подстраиваться
        )
        let item = NSCollectionLayoutItem(layoutSize: itemSize)
        
        let groupSize = NSCollectionLayoutSize(
            widthDimension: .fractionalWidth(0.5),  // 2 колонки
            heightDimension: .estimated(200)
        )
        let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitems: [item])
        group.interItemSpacing = .fixed(8)
        
        let section = NSCollectionLayoutSection(group: group)
        section.interGroupSpacing = 8
        section.contentInsets = NSDirectionalEdgeInsets(top: 8, leading: 8, bottom: 8, trailing: 8)
        
        // Sticky header
        let headerSize = NSCollectionLayoutSize(
            widthDimension: .fractionalWidth(1.0),
            heightDimension: .estimated(44)
        )
        let header = NSCollectionLayoutBoundarySupplementaryItem(
            layoutSize: headerSize,
            elementKind: UICollectionView.elementKindSectionHeader,
            alignment: .top
        )
        header.pinToVisibleBounds = true
        section.boundarySupplementaryItems = [header]
        
        return section
    }
    
    return layout
}
```

#### 3. Смешанные секции (одна коллекция — разные layouts)

```swift
func createMixedLayout() -> UICollectionViewCompositionalLayout {
    return UICollectionViewCompositionalLayout { sectionIndex, environment in
        switch sectionIndex {
        case 0: return createCarouselSection()
        case 1: return createMasonrySection()
        default: return createListSection()
        }
    }
}
```

### Лучшие практики Compositional Layout в Swift 2026

- **Всегда** используйте `estimated` размеры для динамической высоты ячеек — это стандарт  
- **Предпочитайте** `.fractionalWidth` / `.fractionalHeight` — они адаптивны к размеру экрана  
- **Для каруселей** — используйте `.groupPaging`, `.groupPagingCentered`, `.continuous`  
- **Для sticky headers** — `pinToVisibleBounds = true` + `.top` alignment  
- **Для разделителей** — используйте `interItemSpacing` и `interGroupSpacing`  
- **Для кастомных фонов** — добавляйте `NSCollectionLayoutDecorationItem`  
- **В SwiftUI** — используйте `UICollectionView` через `UIViewRepresentable` с compositional layout  
- **Документируйте** — пишите комментарий «UICollectionViewCompositionalLayout — горизонтальная карусель с групповой прокруткой и центрированием»

**Короткий итог 2026**:
> `UICollectionViewCompositionalLayout` — это **современный и единственный рекомендуемый** способ создавать гибкие, адаптивные и сложные коллекции в iOS.  
> В 2026 году:  
> - строится из Item → Group → Section  
> - размеры — fractional / absolute / estimated  
> - поддерживает orthogonal scrolling, sticky headers, nested groups  
> - это **основа** всех современных коллекций (Pinterest, Netflix, App Store, Instagram и т.д.)  

Удачи с красивыми, адаптивными и современными коллекциями в твоём приложении! 📱✨