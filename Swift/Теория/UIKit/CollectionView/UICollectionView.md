**UICollectionView** — это мощный и гибкий компонент UIKit для отображения коллекций данных (сетка, список, карусель, кастомные макеты и т.д.).  
Он используется почти во всех современных iOS-приложениях, где нужно показать много однотипных элементов: фото в галерее, товары в магазине, чаты, профили, рекомендации и т.п.

### Для чего используют UICollectionView в 2026 году (реальные сценарии)

| Сценарий                                   | Почему именно UICollectionView                     | Альтернатива (если не подходит) |
|--------------------------------------------|-----------------------------------------------------|---------------------------------|
| Галерея фото / видео                       | Поддержка кастомных размеров, анимаций, prefetch    | SwiftUI LazyVGrid (если чистый SwiftUI) |
| Магазин / каталог товаров                  | Горизонтальные/вертикальные секции, infinite scroll | SwiftUI LazyHGrid / LazyVGrid |
| Чат (сообщения, сторис)                    | Разные размеры ячеек, динамическая высота          | SwiftUI List / LazyVStack |
| Профили, карточки пользователей            | Кастомные layouts, supplementary views (headers)   | SwiftUI Grid / ScrollView |
| Рекомендации, карусели, onboarding         | Горизонтальный скролл, paging, parallax            | SwiftUI TabView / ScrollView |
| Календарь, планировщик                     | Секции по дням/месяцам, кастомные layouts          | SwiftUI Grid + custom layout |
| Dashboard / виджеты                        | Много разных типов ячеек в одном экране             | SwiftUI LazyVGrid + Section |

### Что необходимо сделать, чтобы начать использовать UICollectionView

Вот **минимальный, но современный** чек-лист на 2026 год (программный подход, без Storyboard).

#### 1. Создать UICollectionView

```swift
let layout = UICollectionViewFlowLayout()
layout.minimumInteritemSpacing = 10
layout.minimumLineSpacing = 10
layout.itemSize = CGSize(width: 150, height: 150) // или dynamic size

let collectionView = UICollectionView(frame: .zero, collectionViewLayout: layout)
collectionView.backgroundColor = .systemBackground
collectionView.translatesAutoresizingMaskIntoConstraints = false
view.addSubview(collectionView)
```

#### 2. Зарегистрировать ячейки (обязательно!)

```swift
// Вариант 1 — кастомная ячейка из xib или кода
collectionView.register(CustomCell.self, forCellWithReuseIdentifier: "CustomCell")

// Вариант 2 — из xib
collectionView.register(UINib(nibName: "CustomCell", bundle: nil), 
                       forCellWithReuseIdentifier: "CustomCell")

// Вариант 3 — стандартная ячейка
collectionView.register(UICollectionViewCell.self, forCellWithReuseIdentifier: "DefaultCell")
```

#### 3. Назначить dataSource и delegate (обычно self)

```swift
collectionView.dataSource = self
collectionView.delegate = self
```

#### 4. Реализовать минимум два метода DataSource

```swift
extension YourViewController: UICollectionViewDataSource {
    
    func collectionView(_ collectionView: UICollectionView, 
                       numberOfItemsInSection section: Int) -> Int {
        return items.count  // или dataSource[section].count
    }
    
    func collectionView(_ collectionView: UICollectionView, 
                       cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "CustomCell", 
                                                     for: indexPath) as! CustomCell
        
        let item = items[indexPath.item]
        cell.configure(with: item)
        
        return cell
    }
}
```

#### 5. (Опционально) Добавить делегат для кликов и других событий

```swift
extension YourViewController: UICollectionViewDelegate {
    func collectionView(_ collectionView: UICollectionView, 
                       didSelectItemAt indexPath: IndexPath) {
        let item = items[indexPath.item]
        // открыть детальный экран, показать алерт и т.д.
    }
}
```

#### 6. (Важно!) Обновлять коллекцию при изменении данных

```swift
// Самый простой способ
collectionView.reloadData()

// Лучше — точечное обновление (анимации + производительность)
collectionView.performBatchUpdates({
    collectionView.insertItems(at: [newIndexPath])
    collectionView.deleteItems(at: [oldIndexPath])
}, completion: nil)

// Или Diffable Data Source (рекомендуется в 2026)
dataSource.apply(snapshot, animatingDifferences: true)
```

### Короткий чек-лист «Что нужно сделать, чтобы UICollectionView заработал»

1. Создать `UICollectionView` + layout  
2. Зарегистрировать все ячейки (обязательно!)  
3. Назначить `dataSource` и `delegate`  
4. Реализовать минимум `numberOfItemsInSection` и `cellForItemAt`  
5. Вызвать `reloadData()` после изменения данных  
6. Добавить constraints или frame (translatesAutoresizingMaskIntoConstraints = false)  
7. (Опционально) Добавить supplementary views (headers/footers) через `register` + `dequeueReusableSupplementaryView`

### Короткий девиз 2026

> UICollectionView — это когда тебе нужна **гибкая сетка** с кастомными размерами ячеек, секциями, анимациями и динамическим контентом.  
> В 2026 году это всё ещё **основной инструмент** для галерей, каталогов, чатов, рекомендаций и любых коллекций в UIKit-приложениях.  
> SwiftUI → LazyVGrid / LazyHGrid, но если нужен полный контроль — UICollectionView.

Удачи с красивой и производительной коллекцией! 📱