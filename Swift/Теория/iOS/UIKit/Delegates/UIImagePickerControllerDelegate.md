**`UIImagePickerControllerDelegate`** — это протокол из [[UIKit]], который определяет методы-делегаты для обработки событий и результатов работы **`UIImagePickerController`**.

Этот протокол **обязателен** при использовании `UIImagePickerController`: без него вы не сможете получить выбранное фото/видео или обработать отмену/ошибку.

### Основные методы протокола (актуально на 2026 год)

| Метод делегата                                      | Когда вызывается                                                                 | Что нужно делать внутри метода                                                                 | Самый частый сценарий |
|-----------------------------------------------------|----------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------|------------------------|
| `imagePickerController(_:didFinishPickingMediaWithInfo:)` | Пользователь выбрал медиа (фото/видео) и нажал "Выбрать"/"Использовать"         | Обязательно вызвать `dismiss(animated:)` и обработать медиа из словаря `info`                  | Получение фото/видео   |
| `imagePickerControllerDidCancel(_:)`                | Пользователь отменил выбор (нажал "Отмена")                                      | Вызвать `dismiss(animated:)` и выполнить действия при отмене                                    | Обработка отмены       |
| `imagePickerController(_:didFinishPickingImage:editingInfo:)` (устаревший) | —                                                                                | Не использовать — заменён на `didFinishPickingMediaWithInfo:`                                  | Legacy-код             |

### Единственный ключевой метод (современный стандарт)

```swift
func imagePickerController(_ picker: UIImagePickerController,
                           didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any])
```

**Самые важные ключи в словаре `info`**

| Ключ (InfoKey)   | Что возвращает                                                     | Тип значения   | Когда доступен                |
| ---------------- | ------------------------------------------------------------------ | -------------- | ----------------------------- |
| `.originalImage` | Исходное изображение (без редактирования)                          | [[UIImage]]`?` | Почти всегда                  |
| `.editedImage`   | Отредактированное изображение (если пользователь обрезал/повернул) | `UIImage?`     | Если разрешено редактирование |
| `.imageURL`      | [[URL]] выбранного фото/видео из библиотеки                        | [[URL]]`?`     | iOS 8+                        |
| `.mediaType`     | Тип медиа ("public.image" / "public.movie")                        | [[String]]`?`  | Всегда                        |
| `.mediaURL`      | URL видео (если выбрано видео)                                     | `URL?`         | Только видео                  |
| `.livePhoto`     | Live Photo (если поддерживается)                                   | `PHLivePhoto?` | iOS 9+                        |
| `.phAsset`       | PHAsset (для доступа к библиотеке Photos)                          | `PHAsset?`     | iOS 11+                       |

### Полный современный пример реализации делегата (рекомендуемый шаблон 2026)

```swift
import UIKit
import Photos

@MainActor
final class ImagePickerHelper: NSObject, UIImagePickerControllerDelegate, UINavigationControllerDelegate {
    
    weak var presentingVC: UIViewController?
    var completion: ((UIImage?, URL?, Error?) -> Void)?
    
    func presentImagePicker(from vc: UIViewController,
                            sourceType: UIImagePickerController.SourceType = .photoLibrary,
                            allowsEditing: Bool = false,
                            completion: @escaping (UIImage?, URL?, Error?) -> Void) {
        
        guard UIImagePickerController.isSourceTypeAvailable(sourceType) else {
            completion(nil, nil, NSError(domain: "ImagePicker", code: -1, userInfo: [NSLocalizedDescriptionKey: "Источник недоступен"]))
            return
        }
        
        self.presentingVC = vc
        self.completion = completion
        
        let picker = UIImagePickerController()
        picker.delegate = self
        picker.sourceType = sourceType
        picker.allowsEditing = allowsEditing
        picker.mediaTypes = ["public.image"] // можно добавить "public.movie" для видео
        
        vc.present(picker, animated: true)
    }
    
    // Обязательный метод делегата
    func imagePickerController(_ picker: UIImagePickerController,
                               didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
        
        picker.dismiss(animated: true)
        
        // Извлекаем изображение
        var image: UIImage?
        if let edited = info[.editedImage] as? UIImage {
            image = edited
        } else if let original = info[.originalImage] as? UIImage {
            image = original
        }
        
        // URL (если выбрано из библиотеки)
        let imageURL = info[.imageURL] as? URL
        
        completion?(image, imageURL, nil)
        completion = nil
        presentingVC = nil
    }
    
    func imagePickerControllerDidCancel(_ picker: UIImagePickerController) {
        picker.dismiss(animated: true)
        completion?(nil, nil, nil)
        completion = nil
        presentingVC = nil
    }
}
```

### Как правильно интегрировать в приложение (рекомендуемый подход)

```swift
class ProfileViewController: UIViewController {
    private let imagePicker = ImagePickerHelper()
    
    @IBAction func changeAvatarTapped() {
        imagePicker.presentImagePicker(from: self) { [weak self] image, url, error in
            guard let self else { return }
            
            if let image {
                self.avatarImageView.image = image
                // Можно сохранить в Documents или Photos
            } else if let error {
                print("Ошибка:", error.localizedDescription)
            }
        }
    }
}
```

### Лучшие практики MFMailComposeViewControllerDelegate в 2026

- **Всегда** проверяйте `UIImagePickerController.isSourceTypeAvailable(_:)` перед показом  
- **Обязательно** реализуйте оба метода делегата  
- **Держите делегат** как отдельный объект (не делайте UIViewController делегатом самого себя — [[retain cycle]])  
- **Используйте** [[@MainActor ]]— все UI-операции и делегаты должны быть на главном потоке  
- **Для SwiftUI** — оборачивайте в `UIViewControllerRepresentable` + `Coordinator`  
- **Для доступа к оригинальному файлу** — используйте `.imageURL` и `.phAsset` (iOS 11+)  
- **Privacy** — добавьте в Info.plist:
  - `NSPhotoLibraryUsageDescription`
  - `NSCameraUsageDescription` (если `.camera`)
- **Документируйте** — пишите комментарий «UIImagePickerControllerDelegate — получение выбранного фото/видео»

**Короткий итог 2026**:
> `UIImagePickerControllerDelegate` — **единственный** способ получить результат выбора фото/видео из `UIImagePickerController`.  
> В 2026 году:  
> - главный метод — `didFinishPickingMediaWithInfo`  
> - всегда обрабатывайте отмену (`didFinishPickingDidCancel`)  
> - используйте как отдельный helper-класс  
> - извлекайте `.originalImage` / `.editedImage` / `.imageURL`  
> Это **стандартный и ожидаемый** пользователем способ выбора медиа в iOS-приложениях.
