**UIDropInteraction** — это механизм в [[UIKit]] (с iOS 11, 2017), который позволяет вашему [[UIView]] (или любому другому view) **принимать перетаскиваемые (drag) объекты** от других частей приложения или из других приложений (drag & drop).

Он отвечает за:

- определение, может ли view принять перетаскиваемый объект в данный момент,
- обработку этапов drop (вход, перемещение, выход, сброс),
- получение данных из перетаскиваемого объекта ([[URL]], строка, изображение, файл и т.д.),
- визуальную обратную связь (подсветка, анимация, изменение курсора).

Это **основной инструмент** для реализации drag & drop внутри приложения и между приложениями в 2026 году.

### Когда использовать UIDropInteraction в 2026 году

| Сценарий                                           | Почему именно UIDropInteraction                    | Альтернатива                                              |
| -------------------------------------------------- | -------------------------------------------------- | --------------------------------------------------------- |
| Перетаскивание файлов из «Файлы» в ваше приложение | Самый надёжный способ импорта файлов               | [[UIDocumentPickerViewController]] (если не нужен drag)   |
| Drag & drop между ячейками таблицы / коллекции     | Полный контроль над позицией, reorder, копирование | [[UITableViewDiffableDataSource]] + reorder (ограниченно) |
| Приём изображений / текста из других приложений    | Поддержка cross-app drag & drop                    | —                                                         |
| Создание плейлиста путём перетаскивания треков     | Нативный UX, как в Apple Music                     | —                                                         |
| Импорт фото из галереи перетаскиванием             | Современный и интуитивный способ                   | [[UIImagePickerController]] (старый стиль)                |

### Основные протоколы и классы

| Компонент                                 | Роль                                                 | Самый важный |
|-------------------------------------------|------------------------------------------------------|--------------|
| `UIDropInteraction`                       | Объект, который добавляется к view через `addInteraction` | Да |
| `UIDropInteractionDelegate`               | Протокол с методами обработки drop                   | Да |
| `UIDropSession`                           | Контекст текущей операции drop (данные, location)    | Да |
| `NSItemProvider`                          | Источник данных (URL, строка, изображение, файл)     | Да |
| `UIDropProposal`                          | Предложение, что произойдёт при сбросе (copy / move / cancel) | Да |

### Самый популярный и рекомендуемый паттерн 2026 года  
(Приём перетаскиваемых изображений / файлов в коллекцию)

```swift
import UIKit

class GalleryCollectionViewController: UICollectionViewController, UIDropInteractionDelegate {
    
    private var images: [UIImage] = []
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // 1. Включаем drop на всю коллекцию
        collectionView.addInteraction(UIDropInteraction(delegate: self))
        collectionView.isUserInteractionEnabled = true
        
        // 2. Регистрируем ячейку
        collectionView.register(UICollectionViewCell.self, forCellWithReuseIdentifier: "cell")
    }
    
    // MARK: - UIDropInteractionDelegate
    
    // Может ли view принять drop в этой точке?
    func dropInteraction(_ interaction: UIDropInteraction, canHandle session: UIDropSession) -> Bool {
        // Принимаем только изображения и файлы изображений
        return session.canLoadObjects(ofClass: UIImage.self) ||
               session.hasItemsConforming(toTypeIdentifiers: ["public.image"])
    }
    
    // Какой эффект показать при перетаскивании над view
    func dropInteraction(_ interaction: UIDropInteraction, sessionDidUpdate session: UIDropSession) -> UIDropProposal {
        let location = session.location(in: collectionView)
        
        // Можно проверить, над какой ячейкой находится курсор
        if let indexPath = collectionView.indexPathForItem(at: location) {
            // Подсветка ячейки (опционально)
        }
        
        // Предлагаем копирование (можно move, если внутри приложения)
        return UIDropProposal(operation: .copy)
    }
    
    // Пользователь сбросил объект — обрабатываем drop
    func dropInteraction(_ interaction: UIDropInteraction, performDrop session: UIDropSession) {
        let location = session.location(in: collectionView)
        
        // Загружаем все перетащенные объекты
        session.loadObjects(ofClass: UIImage.self) { items in
            guard let images = items as? [UIImage] else { return }
            
            // Добавляем изображения в коллекцию
            self.images.append(contentsOf: images)
            
            // Обновляем UI
            DispatchQueue.main.async {
                self.collectionView.reloadData()
            }
        }
        
        // Можно также принимать файлы по URL
        for itemProvider in session.items.map({ $0.itemProvider }) {
            if itemProvider.hasItemConformingToTypeIdentifier("public.image") {
                itemProvider.loadObject(ofClass: UIImage.self) { image, error in
                    guard let image = image as? UIImage, error == nil else { return }
                    DispatchQueue.main.async {
                        self.images.append(image)
                        self.collectionView.reloadData()
                    }
                }
            }
        }
    }
    
    // Визуальная обратная связь при входе/выходе
    func dropInteraction(_ interaction: UIDropInteraction, sessionDidEnter session: UIDropSession) {
        // Подсветка всей коллекции или конкретной ячейки
        collectionView.backgroundColor = .systemGray5.withAlphaComponent(0.5)
    }
    
    func dropInteraction(_ interaction: UIDropInteraction, sessionDidExit session: UIDropSession) {
        collectionView.backgroundColor = .clear
    }
    
    func dropInteraction(_ interaction: UIDropInteraction, sessionDidEnd session: UIDropSession) {
        collectionView.backgroundColor = .clear
    }
}

// MARK: - UICollectionViewDataSource
extension GalleryCollectionViewController {
    override func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        images.count
    }
    
    override func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "cell", for: indexPath)
        // Настройка ячейки с изображением
        return cell
    }
}
```

### Лучшие практики UIDropInteraction в 2026 году

- **Всегда** реализуйте `canHandle(session:)` — это первый фильтр  
- **Возвращайте** правильный `UIDropProposal` — `.copy`, `.move`, `.forbidden`, `.cancel`  
- **Загружайте** данные асинхронно через `loadObjects(ofClass:)` или `loadItem(forTypeIdentifier:)`  
- **Подсвечивайте** область drop через `sessionDidEnter` / `sessionDidExit`  
- **Для SwiftUI** — используйте `.onDrop(of:delegate:)` — UIDropInteraction нужен только в UIKit  
- **Для доступности** — поддерживайте VoiceOver (accessibility) при drop  
- **Документируйте** — пишите комментарий:

```swift
/// UIDropInteraction для приёма изображений и файлов в галерею
collectionView.addInteraction(UIDropInteraction(delegate: self))
```

**Короткий итог 2026**:
> `UIDropInteraction` — механизм для **приёма перетаскиваемых объектов** (drag & drop) в ваш view.  
> В 2026 году:  
> - ключевые методы — `canHandle`, `sessionDidUpdate`, `performDrop`  
> - самый популярный сценарий — импорт файлов, изображений, текста из других приложений  
> - часто используется вместе с `UIDragInteraction` (для двустороннего drag & drop)  
> - в SwiftUI — эквивалент `.onDrop(of:delegate:)`  
> - это **единственный нативный** способ реализовать полноценный drag & drop в UIKit  
