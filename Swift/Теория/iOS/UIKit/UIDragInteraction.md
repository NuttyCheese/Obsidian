**UIDragInteraction** — это механизм в [[UIKit]] (с iOS 11, 2017), который позволяет вашему `UIView` (или любому другому view) **стать источником перетаскиваемого контента** (drag source) в системе drag & drop.

Он отвечает за:

- определение, можно ли начать перетаскивание из этого view,
- создание **перетаскиваемых элементов** (`UIDragItem`),
- предоставление данных (строка, изображение, [[URL]], файл, кастомные объекты),
- визуальную обратную связь при начале drag (поднятие, тень, анимация),
- управление тем, что происходит после сброса (копирование, перемещение, удаление).

Вместе с `UIDropInteraction` (для приёма) это составляет **полноценную систему drag & drop** внутри приложения и между приложениями.

### Когда использовать UIDragInteraction в 2026 году

| Сценарий                                      | Почему именно UIDragInteraction                     | Альтернатива |
|-----------------------------------------------|------------------------------------------------------|--------------|
| Перетаскивание фото из галереи в другое приложение | Нативный drag & drop между приложениями              | UIDocumentInteractionController (ограниченно) |
| Reorder ячеек в коллекции / таблице           | Современный и плавный reorder (вместо старого `canMoveItem`) | UICollectionViewDiffableDataSource + reorder (ограниченно) |
| Перетаскивание текста / ссылок / файлов       | Поддержка cross-app drag (в «Файлы», Сообщения, Заметки) | — |
| Создание плейлиста путём drag треков          | Как в Apple Music / Spotify                          | — |
| Drag кастомных объектов (модель данных)       | Передача структурированных данных между экранами     | Кастомный UIPasteboard (устаревший) |

### Основные протоколы и классы

| Компонент                                 | Роль                                                 | Самый важный |
|-------------------------------------------|------------------------------------------------------|--------------|
| `UIDragInteraction`                       | Объект, который добавляется к view через `addInteraction` | Да |
| `UIDragInteractionDelegate`               | Протокол с методами создания drag items              | Да |
| `UIDragSession`                           | Контекст текущей операции drag                       | Да |
| `UIDragItem`                              | Один перетаскиваемый элемент (может быть несколько)  | Да |
| `NSItemProvider`                          | Источник данных (строка, изображение, URL, файл)     | Да |
| `UIDragPreview` / `UITargetedPreview`     | Визуальный предпросмотр при перетаскивании           | Да |

### Самый популярный и рекомендуемый паттерн 2026 года  
(Перетаскивание изображений / текста из коллекции)

```swift
import UIKit

class GalleryCollectionViewController: UICollectionViewController, UIDragInteractionDelegate {
    
    private var images: [UIImage] = [] // ваши изображения
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // 1. Включаем drag на всю коллекцию
        collectionView.addInteraction(UIDragInteraction(delegate: self))
        collectionView.isUserInteractionEnabled = true
        
        // Регистрируем ячейку
        collectionView.register(UICollectionViewCell.self, forCellWithReuseIdentifier: "cell")
    }
    
    // MARK: - UIDragInteractionDelegate
    
    // Можно ли начать drag из этой точки?
    func dragInteraction(_ interaction: UIDragInteraction, itemsForBeginning session: UIDragSession) -> [UIDragItem] {
        let location = session.location(in: collectionView)
        guard let indexPath = collectionView.indexPathForItem(at: location),
              let cell = collectionView.cellForItem(at: indexPath),
              let image = images[safe: indexPath.item] else {
            return []
        }
        
        // 2. Создаём item provider (источник данных)
        let provider = NSItemProvider(object: image)
        
        // 3. Создаём drag item
        let item = UIDragItem(itemProvider: provider)
        
        // 4. Предпросмотр (очень важно для UX)
        item.previewProvider = { () -> UIDragPreview? in
            let previewImageView = UIImageView(image: image)
            previewImageView.contentMode = .scaleAspectFit
            
            let params = UIDragPreviewParameters()
            params.backgroundColor = .clear
            params.visiblePath = UIBezierPath(roundedRect: previewImageView.bounds, cornerRadius: 12)
            
            let target = UIDragPreviewTarget(container: cell, center: cell.bounds.center)
            return UIDragPreview(view: previewImageView, parameters: params, target: target)
        }
        
        return [item]
    }
    
    // Опционально: предпросмотр при начале drag
    func dragInteraction(_ interaction: UIDragInteraction, previewForCancelling item: UIDragItem) -> UITargetedPreview? {
        let target = UITargetedPreview(view: collectionView)
        return target
    }
    
    // Опционально: эффект после успешного drop
    func dragInteraction(_ interaction: UIDragInteraction, session: UIDragSession, didEndWith operation: UIDropOperation) {
        if operation == .copy || operation == .move {
            print("Элемент успешно перетащен")
        }
    }
}

// MARK: - UICollectionViewDataSource (пример)
extension GalleryCollectionViewController {
    override func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        images.count
    }
    
    override func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "cell", for: indexPath)
        // настройка ячейки с изображением
        return cell
    }
}
```

### Лучшие практики UIDragInteraction в 2026 году

- **Всегда** реализуйте `itemsForBeginning:` — это точка входа  
- **Возвращайте** несколько `UIDragItem`, если перетаскивается несколько объектов  
- **Используйте** `previewProvider` — без красивого предпросмотра UX сильно страдает  
- **Указывайте** правильный `operation` в `UIDropProposal` на стороне drop  
- **Для SwiftUI** — используйте `.draggable(_:)` и `.onDrag { ... }` — UIDragInteraction нужен только в UIKit  
- **Для доступности** — поддерживайте VoiceOver при drag (accessibility)  
- **Документируйте** — пишите комментарий:

```swift
/// UIDragInteraction для перетаскивания изображений из галереи
collectionView.addInteraction(UIDragInteraction(delegate: self))
```

**Короткий итог 2026**:
> `UIDragInteraction` — механизм для **начала перетаскивания** (drag source) из вашего view.  
> В 2026 году:  
> - ключевой метод — `itemsForBeginning:` (возвращает `[UIDragItem]`)  
> - самый популярный сценарий — drag фото, текста, файлов из коллекции/таблицы  
> - часто используется вместе с `UIDropInteraction` (для приёма)  
> - в SwiftUI — эквивалент `.draggable(_:)` и `.onDrag`  
> - это **единственный нативный** способ реализовать полноценный drag в UIKit  
