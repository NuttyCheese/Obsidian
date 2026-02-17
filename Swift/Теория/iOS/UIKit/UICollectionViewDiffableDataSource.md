**UICollectionViewDiffableDataSource** — это современный и рекомендуемый способ управлять данными в [[UICollectionView]] начиная с iOS 13 (2019 год). К 2026 году он стал **де-факто стандартом** для всех новых проектов на [[UIKit]], полностью вытеснив классический [[UICollectionViewDataSource]].

### Почему именно DiffableDataSource, а не старый dataSource

| Сравнение                            | Классический UICollectionViewDataSource     | UICollectionViewDiffableDataSource (2026)  | Победитель   |
| ------------------------------------ | ------------------------------------------- | ------------------------------------------ | ------------ |
| Управление данными                   | Ручное: reloadData(), insert/delete вручную | Снимки (Snapshot) + автоматические diff    | **Diffable** |
| Анимации обновлений                  | Нужно вручную вызывать performBatchUpdates  | Автоматические, плавные, безопасные        | **Diffable** |
| Производительность на больших данных | Хорошая, но легко ошибиться                 | Лучше (diff считает минимальные изменения) | **Diffable** |
| Обработка состояний (loading, empty) | Ручная логика                               | Легко через разные snapshots               | **Diffable** |
| Код и читаемость                     | Много boilerplate                           | Минимум кода, декларативно                 | **Diffable** |
| Поддержка Swift Concurrency          | Требует осторожности                        | Полностью безопасен (+ [[Sendable]])       | **Diffable** |
| Сложность                            | Проще для очень маленьких коллекций         | Чуть сложнее на старте, но окупается       | **Diffable** |

### Самый современный и рекомендуемый паттерн 2026 года

```swift
import UIKit

class PhotosViewController: UIViewController {
    
    // 1. Определяем секции (enum — самый удобный способ)
    enum Section: Hashable {
        case main
    }
    
    // 2. Модель элемента — обязательно Hashable!
    struct Photo: Hashable {
        let id: UUID = UUID()
        let title: String
        let imageName: String
    }
    
    // 3. Типы для data source
    typealias DataSource = UICollectionViewDiffableDataSource<Section, Photo>
    typealias Snapshot = NSDiffableDataSourceSnapshot<Section, Photo>
    
    private var dataSource: DataSource!
    private var collectionView: UICollectionView!
    private var photos: [Photo] = []  // ваш источник данных
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        configureCollectionView()
        configureDataSource()
        loadInitialData()
    }
    
    private func configureCollectionView() {
        let layout = UICollectionViewCompositionalLayout { _, _ in
            let item = NSCollectionLayoutItem(layoutSize: .init(widthDimension: .fractionalWidth(1.0),
                                                                heightDimension: .fractionalHeight(1.0)))
            let group = NSCollectionLayoutGroup.horizontal(layoutSize: .init(widthDimension: .fractionalWidth(1.0),
                                                                             heightDimension: .absolute(150)), subitems: [item])
            group.interItemSpacing = .fixed(10)
            let section = NSCollectionLayoutSection(group: group)
            section.interGroupSpacing = 10
            section.contentInsets = NSDirectionalEdgeInsets(top: 10, leading: 10, bottom: 10, trailing: 10)
            return section
        }
        
        collectionView = UICollectionView(frame: view.bounds, collectionViewLayout: layout)
        collectionView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
        collectionView.backgroundColor = .systemBackground
        view.addSubview(collectionView)
    }
    
    private func configureDataSource() {
        // 4. Создаём data source один раз
        dataSource = UICollectionViewDiffableDataSource(collectionView: collectionView) { collectionView, indexPath, photo in
            let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "PhotoCell", for: indexPath) as! PhotoCell
            cell.configure(with: photo)
            return cell
        }
    }
    
    private func loadInitialData() {
        // 5. Создаём начальный снимок
        var snapshot = Snapshot()
        snapshot.appendSections([.main])
        snapshot.appendItems(photos, toSection: .main)
        
        // Применяем без анимации при первом запуске
        dataSource.apply(snapshot, animatingDifferences: false)
    }
    
    // 6. Как обновлять данные (самое важное!)
    func updatePhotos(_ newPhotos: [Photo]) {
        var snapshot = Snapshot()
        snapshot.appendSections([.main])
        snapshot.appendItems(newPhotos, toSection: .main)
        
        // Применяем с анимацией
        dataSource.apply(snapshot, animatingDifferences: true)
    }
}
```

### Короткий чек-лист «Что нужно сделать, чтобы DiffableDataSource заработал»

1. Определить `Section` (enum Hashable)  
2. Определить модель элемента (struct Hashable)  
3. Создать `typealias DataSource` и `Snapshot`  
4. Зарегистрировать ячейки  
5. Создать dataSource с замыканием cellProvider  
6. Создать snapshot, добавить секции и элементы  
7. Применить snapshot через `apply(_:animatingDifferences:)`

### Короткий девиз 2026

> UICollectionViewDiffableDataSource — это когда ты хочешь **автоматические анимации**, **меньше багов** и **меньше кода** при обновлении коллекции.  
> В 2026 году это **единственный рекомендуемый** способ управлять данными в UICollectionView.  
> Забудь reloadData() и performBatchUpdates() — используй snapshots и apply().
