**UICollectionViewCell** — это базовый класс для создания ячеек в **UICollectionView**.  
Он выполняет роль **контейнера** для вашего контента (картинки, текст, кнопки, градиенты, кастомные вью и т.д.).

В 2026 году это по-прежнему **единственный правильный** способ создавать ячейки для коллекций в UIKit.

### Зачем нужен именно UICollectionViewCell (а не просто UIView)

- Автоматически поддерживает **переиспользование** (reuse) → экономия памяти  
- Имеет встроенный `contentView` — всё UI добавляешь именно туда  
- Поддерживает **выделение**, **выбранное состояние**, **фокус** (focus engine), **accessibility**  
- Работает с **UICollectionViewDiffableDataSource** и **Compositional Layout**  
- Имеет встроенные методы для анимаций (selected/highlighted states)

### Самый современный и рекомендуемый способ в 2026 году

#### 1. Программная ячейка (без xib/storyboard) — самый чистый вариант

```swift
final class PhotoCell: UICollectionViewCell {
    
    // UI-элементы (лениво создаём)
    private lazy var imageView: UIImageView = {
        let iv = UIImageView()
        iv.contentMode = .scaleAspectFill
        iv.clipsToBounds = true
        iv.backgroundColor = .systemGray5
        return iv
    }()
    
    private lazy var titleLabel: UILabel = {
        let label = UILabel()
        label.font = .systemFont(ofSize: 14, weight: .medium)
        label.textColor = .label
        label.textAlignment = .center
        label.numberOfLines = 2
        return label
    }()
    
    // Конфигурация ячейки (вызывается один раз)
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupUI()
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    private func setupUI() {
        contentView.addSubview(imageView)
        contentView.addSubview(titleLabel)
        
        imageView.translatesAutoresizingMaskIntoConstraints = false
        titleLabel.translatesAutoresizingMaskIntoConstraints = false
        
        NSLayoutConstraint.activate([
            imageView.topAnchor.constraint(equalTo: contentView.topAnchor),
            imageView.leadingAnchor.constraint(equalTo: contentView.leadingAnchor),
            imageView.trailingAnchor.constraint(equalTo: contentView.trailingAnchor),
            imageView.heightAnchor.constraint(equalTo: contentView.widthAnchor, multiplier: 1.0),
            
            titleLabel.topAnchor.constraint(equalTo: imageView.bottomAnchor, constant: 8),
            titleLabel.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 8),
            titleLabel.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -8),
            titleLabel.bottomAnchor.constraint(equalTo: contentView.bottomAnchor, constant: -8)
        ])
        
        // Закругление углов ячейки
        contentView.layer.cornerRadius = 12
        contentView.clipsToBounds = true
    }
    
    // Метод конфигурации (вызывается в cellForItemAt)
    func configure(with photo: Photo) {
        imageView.image = photo.image
        titleLabel.text = photo.title
    }
}
```

#### 2. Регистрация ячейки (в viewDidLoad или раньше)

```swift
collectionView.register(PhotoCell.self, forCellWithReuseIdentifier: "PhotoCell")
```

#### 3. Использование в data source

```swift
func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
    let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "PhotoCell", for: indexPath) as! PhotoCell
    let photo = photos[indexPath.item]
    cell.configure(with: photo)
    return cell
}
```

### Лучшие практики UICollectionViewCell в Swift 2026

- **Всегда final class** — экономия vtable, меньше overhead  
- **lazy var** для UI-элементов — создаются только при первом обращении  
- **translatesAutoresizingMaskIntoConstraints = false** — обязательно  
- **contentView** — добавляй все subviews именно туда (не в self)  
- **configure(with:)** — отдельный метод для настройки контента (не в cellForItemAt)  
- **prepareForReuse()** — сбрасывай состояние ячейки (изображения, текст, цвета)

```swift
override func prepareForReuse() {
    super.prepareForReuse()
    imageView.image = nil
    titleLabel.text = nil
}
```

- **@MainActor** — если ячейка обновляет UI из async-кода  
- **Swift 6 strict concurrency** — UICollectionViewCell не Sendable по умолчанию → избегай передачи в Task без @MainActor  
- **Документируйте** — пиши комментарий «UICollectionViewCell — кастомная ячейка для фото с ленивой инициализацией»

**Короткий девиз 2026**:
> UICollectionViewCell — это **контейнер для одной карточки/элемента** в коллекции.  
> Создаёшь подкласс, добавляешь UI в contentView, настраиваешь в configure(with:), регистрируешь один раз.  
> В 2026 году — всегда programmatic + final + lazy + configure-метод.

Удачи с красивыми и производительными ячейками! 📸