#extension #uicollectionview #uicollectionviewcell #uicollectionviewLayout  #protocol

---
Стиль и подход максимально близки к расширению для [[UITableView]]:  
- variadic-регистрация нескольких типов сразу  
- автоматический `reuseIdentifier = String(describing: Type.self)`  
- generic-методы для безопасного dequeue без `as!` (но с [[force unwrap]], как в твоём примере для таблицы)  
- поддержка supplementary views (headers/footers/decorations и т.д.)

```swift
extension UICollectionView {
    /// Регистрирует несколько типов ячеек одновременно
    func registerCells(_ cells: UICollectionViewCell.Type...) {
        cells.forEach { cellType in
            let identifier = String(describing: cellType)
            register(cellType, forCellWithReuseIdentifier: identifier)
        }
    }
    
    /// Получает переиспользуемую ячейку с приведением к нужному типу
    func reuseCell<T: UICollectionViewCell>(_ type: T.Type, for indexPath: IndexPath) -> T {
        let identifier = String(describing: type)
        return dequeueReusableCell(withReuseIdentifier: identifier, for: indexPath) as! T
    }
    
    /// Регистрирует несколько типов supplementary views (headers, footers, custom и т.д.)
    /// kind — например: UICollectionView.elementKindSectionHeader
    func registerSupplementaryViews(
        _ types: UICollectionReusableView.Type...,
        for kind: String
    ) {
        types.forEach { viewType in
            let identifier = String(describing: viewType)
            register(viewType, forSupplementaryViewOfKind: kind, withReuseIdentifier: identifier)
        }
    }
    
    /// Получает переиспользуемый supplementary view с приведением к типу
    func reuseSupplementary<T: UICollectionReusableView>(
        _ type: T.Type,
        kind: String,
        at indexPath: IndexPath
    ) -> T {
        let identifier = String(describing: type)
        return dequeueReusableSupplementaryView(
            ofKind: kind,
            withReuseIdentifier: identifier,
            for: indexPath
        ) as! T
    }
}
```

### Варианты использования (самые популярные паттерны 2025–2026)

#### 1. Регистрация в `viewDidLoad` / `awakeFromNib`

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    collectionView.registerCells(
        PhotoCell.self,
        StoryCell.self,
        AdBannerCell.self,
        LoadingCell.self
    )
    
    // Для хедеров
    collectionView.registerSupplementaryViews(
        SectionHeaderView.self,
        CategoryHeader.self,
        for: UICollectionView.elementKindSectionHeader
    )
    
    // Для футеров (если нужны)
    collectionView.registerSupplementaryViews(
        LoadMoreFooter.self,
        for: UICollectionView.elementKindSectionFooter
    )
    
    // Для кастомных decoration views (редко, но бывает)
    collectionView.registerSupplementaryViews(
        BackgroundDecorationView.self,
        for: "background-decoration"
    )
}
```

#### 2. В `cellForItemAt` и `viewForSupplementaryElementOfKind`

```swift
func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
    let item = items[indexPath.item]
    
    if item.isAd {
        let cell = collectionView.reuseCell(AdBannerCell.self, for: indexPath)
        cell.configure(with: item.adData)
        return cell
    } else {
        return collectionView.reuseCell(PhotoCell.self, for: indexPath)
            .configure(with: item.photo)
    }
}

func collectionView(_ collectionView: UICollectionView,
                    viewForSupplementaryElementOfKind kind: String,
                    at indexPath: IndexPath) -> UICollectionReusableView {
    
    switch kind {
    case UICollectionView.elementKindSectionHeader:
        return collectionView.reuseSupplementary(
            SectionHeaderView.self,
            kind: kind,
            at: indexPath
        )
        .setTitle(sectionTitles[indexPath.section])
        
    case UICollectionView.elementKindSectionFooter:
        return collectionView.reuseSupplementary(
            LoadMoreFooter.self,
            kind: kind,
            at: indexPath
        )
        
    default:
        return UICollectionReusableView()
    }
}
```

### Рекомендуемые улучшения (что часто добавляют в 2025+)

1. Более безопасная версия dequeue (с fatalError вместо краша на nil)

```swift
func dequeue<T: UICollectionViewCell>(_ type: T.Type, for indexPath: IndexPath) -> T {
    let id = String(describing: type)
    guard let cell = dequeueReusableCell(withReuseIdentifier: id, for: indexPath) as? T else {
        fatalError("Не удалось dequeue \(id). Зарегистрирована ли ячейка?")
    }
    return cell
}
```

2. Короткий алиас (очень популярен)

```swift
func cell<T: UICollectionViewCell>(ofType: T.Type, at indexPath: IndexPath) -> T {
    reuseCell(ofType, for: indexPath)
}
```

3. Поддержка nib-based ячеек (если используешь XIB)

```swift
func registerCellNibs(_ nibNames: String...) {
    nibNames.forEach { nibName in
        let nib = UINib(nibName: nibName, bundle: nil)
        register(nib, forCellWithReuseIdentifier: nibName)
    }
}
```

### Сравнение со старым стилем

| Задача                          | Классический способ                                      | С extension                                      |
|---------------------------------|----------------------------------------------------------|--------------------------------------------------|
| Регистрация 4 ячеек             | 4 строки register                                        | 1 строка `registerCells(...)`                    |
| Получение ячейки                | `as! PhotoCell` или guard + as?                          | `reuseCell(PhotoCell.self, for: indexPath)`      |
| Регистрация хедеров             | несколько register(forSupplementaryViewOfKind:)          | `registerSupplementaryViews(..., for: kind)`     |
| Получение supplementary         | dequeue + as! / as?                                      | `reuseSupplementary(..., kind: ..., at:)`        |

Если хочешь — можно добавить ещё вариации:  
- поддержку compositional layout + CellRegistration (iOS 14+)  
- автоматическую регистрацию nib/class по протоколу Reusable  
- методы для reloadSections / performBatchUpdates с анимацией

Кидай следующий кусок кода, если есть — продолжим в том же стиле! 🚀