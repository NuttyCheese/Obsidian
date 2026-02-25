**UIImagePickerController** — это системный контроллер Apple для выбора и съёмки медиа (фото и видео) из следующих источников:

- Фото/видео из **библиотеки** (Camera Roll / Photos)
- Съёмка **нового фото** или **видео** с камеры
- Выбор из **альбомов** (включая iCloud, Recently Deleted и т.д.)

Он существует с [[iOS]] 2.0 и по состоянию на 2026 год остаётся **единственным официальным способом** получить доступ к камере и галерее в **нативном** стиле Apple.

### Основные возможности и ограничения (2026)

| Возможность                              | Поддержка в 2026 | Примечание |
|------------------------------------------|------------------|------------|
| Выбор фото из галереи                    | Да               | Основной сценарий |
| Съёмка нового фото                       | Да               | sourceType = .camera |
| Съёмка видео                             | Да               | sourceType = .camera + mediaTypes |
| Выбор видео из галереи                   | Да               | mediaTypes = [.movie] |
| Редактирование после выбора (crop, rotate) | Да (только для фото) | allowsEditing = true |
| Live Photos                              | Да               | автоматически включается |
| Выбор нескольких фото (multi-select)     | **Нет**          | Системный picker не поддерживает множественный выбор |
| Доступ к камере в фоне                   | Нет              | Требуется foreground |
| Доступ без запроса разрешения            | Нет              | Требуется запрос NSCameraUsageDescription / NSPhotoLibraryUsageDescription |
| Поддержка ProRAW / ProRes                | Частично         | Только если устройство поддерживает |

### Ключевые Info.plist ключи (обязательно с iOS 14+)

```xml
<!-- Для доступа к камере -->
<key>NSCameraUsageDescription</key>
<string>Приложению нужен доступ к камере, чтобы вы могли сделать фото профиля</string>

<!-- Для доступа к галерее (выбор фото/видео) -->
<key>NSPhotoLibraryUsageDescription</key>
<string>Приложению нужен доступ к вашей фотобиблиотеке, чтобы вы могли выбрать фото</string>

<!-- Для записи видео -->
<key>NSMicrophoneUsageDescription</key>
<string>Приложению нужен микрофон для записи видео</string>
```

### Самый популярный и рекомендуемый паттерн 2026 года ([[UIKit]] + [[async]]/[[await]])

```swift
import UIKit

class ProfileViewController: UIViewController, UIImagePickerControllerDelegate, UINavigationControllerDelegate {
    
    private let imageView = UIImageView()
    private var imagePicker: UIImagePickerController?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
    }
    
    private func setupUI() {
        imageView.contentMode = .scaleAspectFill
        imageView.clipsToBounds = true
        imageView.backgroundColor = .systemGray5
        imageView.layer.cornerRadius = 16
        imageView.isUserInteractionEnabled = true
        
        let tap = UITapGestureRecognizer(target: self, action: #selector(openImagePicker))
        imageView.addGestureRecognizer(tap)
        
        // добавьте imageView в иерархию через constraints
    }
    
    @objc private func openImagePicker() {
        Task { @MainActor in
            await showImagePicker()
        }
    }
    
    private func showImagePicker() async {
        // Проверка разрешения на фото
        let status = await PHPhotoLibrary.requestAuthorization(for: .addOnly)
        guard status == .authorized || status == .limited else {
            showPermissionDeniedAlert()
            return
        }
        
        let picker = UIImagePickerController()
        picker.delegate = self
        picker.allowsEditing = true                     // включить кроп/поворот
        picker.sourceType = .photoLibrary               // или .camera
        picker.mediaTypes = UIImagePickerController.availableMediaTypes(for: .photoLibrary) ?? []
        
        // Современные настройки (iOS 14+)
        picker.modalPresentationStyle = .fullScreen
        picker.modalTransitionStyle = .coverVertical
        
        self.imagePicker = picker
        present(picker, animated: true)
    }
    
    // MARK: - UIImagePickerControllerDelegate
    
    func imagePickerController(_ picker: UIImagePickerController,
                               didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
        
        picker.dismiss(animated: true)
        
        // Получаем отредактированное изображение (если allowsEditing = true)
        if let editedImage = info[.editedImage] as? UIImage {
            imageView.image = editedImage
            // Сохраняем или загружаем на сервер
            saveImage(editedImage)
        }
        // или оригинальное, если редактирование не включено
        else if let originalImage = info[.originalImage] as? UIImage {
            imageView.image = originalImage
        }
        
        self.imagePicker = nil
    }
    
    func imagePickerControllerDidCancel(_ picker: UIImagePickerController) {
        picker.dismiss(animated: true)
        self.imagePicker = nil
    }
    
    private func saveImage(_ image: UIImage) {
        // Пример: сохранение в Documents или отправка на сервер
        if let data = image.jpegData(compressionQuality: 0.8) {
            // сохранение...
        }
    }
    
    private func showPermissionDeniedAlert() {
        let alert = UIAlertController(
            title: "Доступ к фото запрещён",
            message: "Разрешите доступ в настройках → Конфиденциальность → Фото",
            preferredStyle: .alert
        )
        alert.addAction(UIAlertAction(title: "Настройки", style: .default) { _ in
            if let url = URL(string: UIApplication.openSettingsURLString) {
                UIApplication.shared.open(url)
            }
        })
        alert.addAction(UIAlertAction(title: "Отмена", style: .cancel))
        present(alert, animated: true)
    }
}
```

### Современные альтернативы и тренды 2026 года

| Альтернатива                                | Когда использовать в 2026                                             | Плюсы                                                                                 | Минусы                            |
| ------------------------------------------- | --------------------------------------------------------------------- | ------------------------------------------------------------------------------------- | --------------------------------- |
| **[[PHPickerViewController]]** (iOS 14+)    | Почти всегда вместо UIImagePickerController                           | Поддержка multi-select, ограниченный доступ, не требует full photo library permission | Нет съёмки камерой                |
| **[[UIImagePickerController]]**             | Только если нужна именно **камера** или совместимость с iOS 13 и ниже | Простота, съёмка, редактирование                                                      | Требует полного доступа к галерее |
| **SwiftUI PhotosPicker** (iOS 16+)          | В чистых SwiftUI-приложениях                                          | Нативный, multi-select, безопасный                                                    | Нет камеры                        |
| **[[AVCaptureSession]]** + кастомная камера | Когда нужен полный контроль над камерой                               | Максимальная кастомизация                                                             | Очень много кода                  |

**Рекомендация 2026**:  
Если вам **не нужна камера** → используйте **PHPickerViewController** (или PhotosPicker в SwiftUI).  
Если нужна **камера** → всё ещё используйте **UIImagePickerController**.

### Лучшие практики UIImagePickerController в 2026

- **Предпочитайте** **PHPickerViewController** для выбора из галереи — безопаснее и современнее  
- **Используйте** `allowsEditing = true` — даёт встроенный кроп/поворот  
- **Всегда** проверяйте разрешения перед показом (`PHPhotoLibrary.requestAuthorization`)  
- **Для [[SwiftUI]]** — используйте `PhotosPicker` (iOS 16+) — проще и не требует делегата  
- **Для камеры** — `sourceType = .camera`, но учитывайте, что нужен **NSCameraUsageDescription**  
- **Privacy Manifest** — обязателен с iOS 17+ при использовании UIImagePickerController  
- **Документируйте** — пишите комментарий «UIImagePickerController — выбор фото из галереи с возможностью редактирования (allowsEditing = true)»

**Короткий итог 2026**:
> UIImagePickerController — это **системный контроллер** для выбора/съёмки фото и видео.  
> В 2026 году:  
> - главный сценарий — `sourceType = .photoLibrary` или `.camera`  
> - позволяет редактирование (`allowsEditing = true`)  
> - требует Info.plist ключи и проверки разрешений  
> - для выбора из галереи лучше использовать **PHPickerViewController**  
> Это **классический**, но уже **не самый предпочтительный** способ работы с медиа в iOS.
