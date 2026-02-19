**UICollectionViewController** — это специализированный подкласс **[[UIViewController]]** из UIKit, который уже имеет встроенную поддержку **[[UICollectionView]]**.

Он предназначен для упрощения создания экранов, где **основной контент** — это коллекция (сетка, список, карусель и т.д.).

### Основные особенности UICollectionViewController (актуально на 2026 год)

| Характеристика                                      | Описание                                                                          | Преимущества / Когда использовать в 2026         |
| --------------------------------------------------- | --------------------------------------------------------------------------------- | ------------------------------------------------ |
| Встроенная `collectionView`                         | Свойство `collectionView: UICollectionView!` создаётся автоматически              | Не нужно вручную создавать и добавлять коллекцию |
| Автоматическое назначение dataSource и [[Delegate]] | `self` (контроллер) становится dataSource и delegate по умолчанию                 | Меньше boilerplate кода                          |
| Поддержка **storyboard / [[xib]]**                  | Можно подключить коллекцию прямо в Interface Builder                              | Удобно для простых экранов                       |
| Поддержка **программного создания**                 | Можно переопределить `loadView()` или использовать [[init]]                       | Полный контроль в коде                           |
| Полная совместимость с **DiffableDataSource**       | Работает с [[UICollectionViewDiffableDataSource]] без проблем                     | Стандарт 2026 года                               |
| Жизненный цикл                                      | Те же методы, что у UIViewController ([[viewDidLoad]], [[viewWillAppear]] и т.д.) | Никаких сюрпризов                                |

### Когда использовать UICollectionViewController в 2026 году

**Да, стоит использовать**, если:

- Экран **почти полностью состоит** из UICollectionView (галерея, каталог, чат, рекомендации)  
- Хочешь **минимум boilerplate** (не писать вручную addSubview, constraints)  
- Работаешь с **UIKit** (Storyboard, [[xib]], programmatic [[UIKit]])  
- Нужна **быстрая разработка** простых коллекций

**Нет, лучше взять обычный UIViewController**, если:

- В экране **много другого UI** (header, footer, кнопки, поисковая строка, navigation bar custom)  
- Используешь **SwiftUI** — там `UICollectionViewController` не нужен  
- Хочешь **полный контроль** над иерархией вью (например, collectionView — не главный view)  
- Используешь **[[TCA]] / [[MVVM (Model-View-ViewModel) Architecture|MVVM]]-[[Coordinator]] / [[VIPER Architecture|VIPER]]** — обычно предпочитают обычный контроллер + ViewModel

### Самый современный и рекомендуемый паттерн 2026 года

#### Программный подход (без Storyboard)

```swift
final class PhotosCollectionViewController: UICollectionViewController {
    
    private var photos: [Photo] = []  // ваш массив данных
    
    // 1. Инициализация с кастомным layout
    init() {
        let layout = UICollectionViewFlowLayout()
        layout.minimumInteritemSpacing = 10
        layout.minimumLineSpacing = 10
        layout.sectionInset = UIEdgeInsets(top: 16, left: 16, bottom: 16, right: 16)
        layout.itemSize = CGSize(width: 150, height: 150) // или dynamic
        
        super.init(collectionViewLayout: layout)
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // 2. Настройка коллекции (уже существует!)
        collectionView.backgroundColor = .systemBackground
        collectionView.alwaysBounceVertical = true
        
        // 3. Регистрация ячейки
        collectionView.register(PhotoCell.self, forCellWithReuseIdentifier: "PhotoCell")
        
        // 4. Загрузка данных
        loadPhotos()
    }
    
    private func loadPhotos() {
        Task {
            do {
                photos = try await fetchPhotos()
                await MainActor.run {
                    collectionView.reloadData()
                }
            } catch {
                // обработка ошибки
            }
        }
    }
}

// MARK: - UICollectionViewDataSource
extension PhotosCollectionViewController {
    override func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return photos.count
    }
    
    override func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "PhotoCell", for: indexPath) as! PhotoCell
        let photo = photos[indexPath.item]
        cell.configure(with: photo)
        return cell
    }
}
```

### Лучшие практики UICollectionViewController в Swift 2026

- **override init(collectionViewLayout:)** — основной способ создания  
- **Не забывай super.init(collectionViewLayout:)** — иначе коллекция не инициализируется  
- **collectionView** — уже существует и настроено, используй его напрямую  
- **[[@MainActor]]** — весь контроллер — на главном акторе  
- **DiffableDataSource** — **обязательно** для новых проектов (анимации, производительность)  
- **Swift 6 strict concurrency** — UICollectionViewController полностью безопасен (@MainActor)  
- **Документируйте** — пиши комментарий «UICollectionViewController — экран с коллекцией фото»

**Короткий девиз 2026**:
> UICollectionViewController — это когда тебе нужен **готовый контроллер с коллекцией внутри**, минимум boilerplate и максимальная скорость разработки.  
> В 2026 году это **отличный выбор** для экранов, где 90%+ контента — это UICollectionView.  
> Если экран сложный (header, footer, кнопки, поиск) — лучше обычный [[UIViewController]] + collectionView как subview.
