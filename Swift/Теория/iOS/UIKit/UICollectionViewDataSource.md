**UICollectionViewDataSource** — это протокол в [[UIKit]], который отвечает за **предоставление данных** коллекционному представлению ([[UICollectionView]]).

Он говорит коллекции:
- сколько у неё секций
- сколько элементов в каждой секции
- какую ячейку показать для каждого индекса

Это **обязательный** протокол — без него коллекция просто не сможет отобразить ничего.

### Обязательные методы (must implement)

| Метод                                              | Что возвращает                                 | Самый частый пример реализации 2026 |
|----------------------------------------------------|------------------------------------------------|-------------------------------------|
| `collectionView(_:numberOfItemsInSection:)`        | Количество элементов в конкретной секции       | `return items.count` или `data[section].count` |
| `collectionView(_:cellForItemAt:)`                 | Готовую ячейку для показа по indexPath         | `dequeueReusableCell` → `configure(with:)` → `return cell` |

### Очень часто реализуемые (но необязательные) методы

| Метод                                              | Что делает                                     | Когда обычно нужен |
|----------------------------------------------------|------------------------------------------------|---------------------|
| `numberOfSections(in:)`                            | Сколько секций всего                           | Почти всегда (по умолчанию 1) |
| `collectionView(_:viewForSupplementaryElementOfKind:at:)` | Заголовки, футеры, decoration views            | При использовании headers/footers |
| `collectionView(_:canMoveItemAt:)`                 | Можно ли перетаскивать элемент                 | Drag & drop поддержка |
| `collectionView(_:moveItemAt:to:)`                 | Обработка перемещения элемента                 | Drag & drop |

### Самый современный и рекомендуемый паттерн 2026 года

#### Вариант 1 — Классический dataSource (для простых коллекций)

```swift
class PhotosViewController: UIViewController {
    
    private var photos: [Photo] = []
    private let collectionView = UICollectionView(frame: .zero, collectionViewLayout: UICollectionViewFlowLayout())
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        collectionView.dataSource = self
        collectionView.delegate = self  // если нужен didSelect и т.д.
        collectionView.register(PhotoCell.self, forCellWithReuseIdentifier: "PhotoCell")
        
        // ... constraints для collectionView
    }
}

extension PhotosViewController: UICollectionViewDataSource {
    
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return photos.count
    }
    
    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "PhotoCell", for: indexPath) as! PhotoCell
        let photo = photos[indexPath.item]
        cell.configure(with: photo)
        return cell
    }
}
```

#### Вариант 2 — Самый рекомендуемый в 2026: **UICollectionViewDiffableDataSource** (снимки + анимации)

```swift
class PhotosViewController: UIViewController {
    
    enum Section {
        case main
    }
    
    private typealias DataSource = UICollectionViewDiffableDataSource<Section, Photo>
    private typealias Snapshot = NSDiffableDataSourceSnapshot<Section, Photo>
    
    private var dataSource: DataSource!
    private var collectionView: UICollectionView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        configureCollectionView()
        configureDataSource()
        applyInitialSnapshot()
    }
    
    private func configureCollectionView() {
        let layout = UICollectionViewCompositionalLayout { _, _ in
            let item = NSCollectionLayoutItem(layoutSize: .init(widthDimension: .fractionalWidth(1.0),
                                                                heightDimension: .fractionalHeight(1.0)))
            let group = NSCollectionLayoutGroup.horizontal(layoutSize: .init(widthDimension: .fractionalWidth(1.0),
                                                                             heightDimension: .absolute(150)), subitems: [item])
            return NSCollectionLayoutSection(group: group)
        }
        
        collectionView = UICollectionView(frame: view.bounds, collectionViewLayout: layout)
        collectionView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
        view.addSubview(collectionView)
    }
    
    private func configureDataSource() {
        dataSource = UICollectionViewDiffableDataSource(collectionView: collectionView) { collectionView, indexPath, photo in
            let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "PhotoCell", for: indexPath) as! PhotoCell
            cell.configure(with: photo)
            return cell
        }
    }
    
    private func applyInitialSnapshot() {
        var snapshot = Snapshot()
        snapshot.appendSections([.main])
        snapshot.appendItems(photos, toSection: .main)
        dataSource.apply(snapshot, animatingDifferences: false)
    }
    
    // При обновлении данных
    func updatePhotos(_ newPhotos: [Photo]) {
        var snapshot = Snapshot()
        snapshot.appendSections([.main])
        snapshot.appendItems(newPhotos, toSection: .main)
        dataSource.apply(snapshot, animatingDifferences: true)
    }
}
```

### Короткий чек-лист «Что нужно сделать, чтобы UICollectionViewDataSource заработал»

1. Создать [[UICollectionView]]  
2. Зарегистрировать ячейки (обязательно!)  
3. Назначить dataSource ([[self]] или отдельный объект)  
4. Реализовать минимум два метода:
   - `numberOfItemsInSection`  
   - `cellForItemAt`  
5. Вызвать `reloadData()` или применить snapshot при изменении данных  

### Короткий девиз 2026

> UICollectionViewDataSource — это когда ты говоришь коллекции:  
> «сколько у меня элементов, и какую ячейку показать на этом месте».  
> В 2026 году **рекомендуется** использовать **DiffableDataSource** — он даёт анимации, производительность и меньше багов.
