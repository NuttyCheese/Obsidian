**UICollectionViewDelegate** — это протокол в [[UIKit]], который отвечает за **взаимодействие пользователя** с коллекцией и **управление её поведением**.

Он дополняет **[[UICollectionViewDataSource]]** (который только даёт данные) и обрабатывает:

- выбор/выделение ячеек  
- размеры ячеек (если используешь Flow Layout)  
- подсветку / unhighlight  
- появление/исчезновение ячеек  
- скролл, перетаскивание, фокус и многое другое

### Основные методы протокола (самые важные в 2026 году)

| Метод                                              | Когда вызывается                                  | Самый частый пример использования 2026 |
|----------------------------------------------------|---------------------------------------------------|----------------------------------------|
| `collectionView(_:didSelectItemAt:)`               | Пользователь нажал на ячейку                      | Открыть детальный экран, показать алерт |
| `collectionView(_:didDeselectItemAt:)`             | Ячейка потеряла выделение                         | Снять выделение, обновить UI           |
| `collectionView(_:didHighlightItemAt:)` / `didUnhighlightItemAt:` | Ячейка нажата / отпущена (touch down/up)          | Анимация нажатия (scale, alpha)        |
| `collectionView(_:shouldSelectItemAt:)` / `shouldDeselectItemAt:` | Можно ли вообще выбрать/снять выделение           | Запретить выбор определённых ячеек     |
| `collectionView(_:layout:sizeForItemAt:)`          | Запрос размера ячейки (только для Flow Layout)    | Динамическая высота/ширина             |
| `collectionView(_:willDisplay:forItemAt:)`         | Ячейка вот-вот появится на экране                 | Prefetch изображений, анимации входа   |
| `collectionView(_:didEndDisplaying:forItemAt:)`    | Ячейка ушла с экрана                              | Отменить загрузку изображений          |
| `scrollViewDidScroll(_:)` (из UIScrollViewDelegate) | Прокрутка коллекции                               | Parallax, sticky headers               |

### Самый современный и рекомендуемый паттерн 2026 года

```swift
@MainActor
class PhotosViewController: UIViewController {
    
    private var collectionView: UICollectionView!
    private var photos: [Photo] = []
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let layout = UICollectionViewFlowLayout()
        layout.minimumInteritemSpacing = 10
        layout.minimumLineSpacing = 10
        
        collectionView = UICollectionView(frame: .zero, collectionViewLayout: layout)
        collectionView.backgroundColor = .systemBackground
        collectionView.dataSource = self
        collectionView.delegate = self
        collectionView.register(PhotoCell.self, forCellWithReuseIdentifier: "PhotoCell")
        
        view.addSubview(collectionView)
        // constraints...
    }
}

extension PhotosViewController: UICollectionViewDelegate {
    
    func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
        let photo = photos[indexPath.item]
        // Открыть полноэкранный просмотр
        let detailVC = PhotoDetailViewController(photo: photo)
        navigationController?.pushViewController(detailVC, animated: true)
    }
    
    func collectionView(_ collectionView: UICollectionView, 
                       didHighlightItemAt indexPath: IndexPath) {
        if let cell = collectionView.cellForItem(at: indexPath) {
            UIView.animate(withDuration: 0.15) {
                cell.transform = CGAffineTransform(scaleX: 0.95, y: 0.95)
                cell.contentView.alpha = 0.8
            }
        }
    }
    
    func collectionView(_ collectionView: UICollectionView, 
                       didUnhighlightItemAt indexPath: IndexPath) {
        if let cell = collectionView.cellForItem(at: indexPath) {
            UIView.animate(withDuration: 0.15) {
                cell.transform = .identity
                cell.contentView.alpha = 1.0
            }
        }
    }
    
    // Динамический размер ячейки (если layout — Flow)
    func collectionView(_ collectionView: UICollectionView, 
                       layout collectionViewLayout: UICollectionViewLayout, 
                       sizeForItemAt indexPath: IndexPath) -> CGSize {
        let width = (collectionView.bounds.width - 30) / 2  // 2 колонки + отступы
        return CGSize(width: width, height: width * 1.5)   // соотношение 2:3
    }
}
```

### Лучшие практики UICollectionViewDelegate в Swift 2026

- **delegate = self** — чаще всего контроллер сам себе делегат  
- **didSelectItemAt** — основной метод для обработки нажатия  
- **didHighlight / didUnhighlight** — для красивой анимации нажатия (scale 0.95, alpha 0.8)  
- **shouldSelectItemAt** — если нужно запретить выбор некоторых ячеек  
- **willDisplay / didEndDisplaying** — для prefetch изображений / отмены загрузки  
- **[[@MainActor]]** — весь контроллер или методы делегата — на главном акторе  
- **Swift 6 strict concurrency** — UICollectionViewDelegate методы вызываются на главном потоке → безопасно  
- **Документируйте** — пиши комментарий «UICollectionViewDelegate — обработка выбора и анимации нажатия»

**Короткий девиз 2026**:
> UICollectionViewDelegate — это когда ты говоришь коллекции:  
> «что делать, когда пользователь нажал ячейку, подсветил её, начал скроллить или она появилась/исчезла с экрана».  
> В 2026 году основные методы — didSelect, didHighlight/didUnhighlight и sizeForItemAt (для Flow Layout).  
> Всё остальное — в DiffableDataSource или Compositional Layout.
