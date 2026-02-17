**UICollectionViewFlowLayout** — это самый популярный и самый часто используемый **стандартный макет** для `UICollectionView` в UIKit.

Он автоматически размещает ячейки в **сетке** (вертикальной или горизонтальной) с фиксированными/динамическими размерами, отступами между ячейками и строками, заголовками/футерами и поддержкой скролла.

В 2026 году это всё ещё **основной выбор**, если не нужен очень сложный кастомный layout (в этом случае берут **UICollectionViewCompositionalLayout**).

### Ключевые свойства и настройки (самые важные в 2026)

| Свойство / Метод                              | Тип / Значение по умолчанию                     | Что делает / Когда менять                              | Рекомендация 2026 |
|-----------------------------------------------|--------------------------------------------------|---------------------------------------------------------|-------------------|
| `itemSize`                                    | CGSize (по умолчанию 50×50)                      | Фиксированный размер всех ячеек                         | Используй для простой сетки |
| `minimumInteritemSpacing`                     | CGFloat (по умолчанию 10)                        | Минимальное расстояние между ячейками в одной строке    | 8–16 для красоты |
| `minimumLineSpacing`                          | CGFloat (по умолчанию 10)                        | Минимальное расстояние между строками                  | 8–20 |
| `sectionInset`                                | UIEdgeInsets (по умолчанию .zero)                | Отступы от краёв секции (padding)                       | 10–20 со всех сторон |
| `scrollDirection`                             | .vertical (по умолчанию)                         | .horizontal — горизонтальный скролл                     | .horizontal для каруселей |
| `headerReferenceSize` / `footerReferenceSize` | CGSize.zero                                      | Размер заголовка/футера секции                          | 44–100 для заголовков |
| `estimatedItemSize`                           | CGSize.zero                                      | Динамический размер ячеек (авто-высота)                 | Используй для динамической высоты |
| `sectionHeadersPinToVisibleBounds`            | false                                            | Закреплять заголовки при скролле                        | true для sticky headers |

### Самый современный и рекомендуемый паттерн 2026 года

```swift
final class PhotosViewController: UIViewController {
    
    private lazy var collectionView: UICollectionView = {
        let layout = UICollectionViewFlowLayout()
        layout.scrollDirection = .vertical
        layout.minimumInteritemSpacing = 10
        layout.minimumLineSpacing = 10
        layout.sectionInset = UIEdgeInsets(top: 16, left: 16, bottom: 16, right: 16)
        
        // Динамический размер ячеек (рекомендуется в 2026)
        layout.estimatedItemSize = UICollectionViewFlowLayout.automaticSize
        
        let cv = UICollectionView(frame: .zero, collectionViewLayout: layout)
        cv.backgroundColor = .systemBackground
        cv.dataSource = self
        cv.delegate = self
        cv.register(PhotoCell.self, forCellWithReuseIdentifier: "PhotoCell")
        return cv
    }()
    
    private var photos: [Photo] = []  // ваш массив данных
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.addSubview(collectionView)
        
        // Constraints (программный Auto Layout)
        collectionView.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            collectionView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            collectionView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            collectionView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            collectionView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
        
        // Загрузка данных
        loadPhotos()
    }
    
    private func loadPhotos() {
        // ... загрузка из сети или локально
        collectionView.reloadData()  // или лучше DiffableDataSource
    }
}

// MARK: - UICollectionViewDataSource
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

// MARK: - UICollectionViewDelegateFlowLayout (опционально)
extension PhotosViewController: UICollectionViewDelegateFlowLayout {
    func collectionView(_ collectionView: UICollectionView, 
                       layout collectionViewLayout: UICollectionViewLayout, 
                       sizeForItemAt indexPath: IndexPath) -> CGSize {
        let width = (collectionView.bounds.width - 42) / 2  // 2 колонки + отступы
        return CGSize(width: width, height: width * 1.5)   // соотношение 2:3
    }
}
```

### Лучшие практики UICollectionViewFlowLayout в Swift 2026

- **estimatedItemSize** — используй для динамической высоты/ширины ячеек  
- **sectionInset** — добавляй отступы от краёв секции (10–20 обычно идеально)  
- **minimumInteritemSpacing / minimumLineSpacing** — 8–16 для красивого вида  
- **scrollDirection = .horizontal** — для каруселей, сторис, рекомендаций  
- **headerReferenceSize** — для sticky заголовков секций  
- **@MainActor** — весь контроллер — на главном акторе  
- **Swift 6 strict concurrency** — UICollectionViewFlowLayout полностью безопасен  
- **Документируйте** — пиши комментарий «UICollectionViewFlowLayout — вертикальная сетка 2 колонки с динамическим размером»

**Короткий девиз 2026**:
> UICollectionViewFlowLayout — это когда тебе нужна **простая, но красивая сетка** или горизонтальная карусель без сложных кастомных макетов.  
> В 2026 году это **основной** layout для большинства коллекций.  
> Если нужна очень сложная структура — переходи на **UICollectionViewCompositionalLayout** или **SwiftUI LazyVGrid**.

Удачи с красивой и отзывчивой коллекцией! 📱