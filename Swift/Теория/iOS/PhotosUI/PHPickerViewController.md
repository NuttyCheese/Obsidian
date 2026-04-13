**PHPickerViewController** — это современный контроллер выбора фото и видео из галереи, представленный Apple в **iOS 14** (2020 год).

Он пришёл на смену `UIImagePickerController` для сценариев **только выбора медиа** (без съёмки камерой) и является **рекомендуемым** способом в 2025–2026 годах.

### Почему PHPickerViewController лучше [[UIImagePickerController]] в 2026 году

| Критерий                              | UIImagePickerController (старый)                  | PHPickerViewController (новый)                          | Выигрыш в 2026 |
|---------------------------------------|----------------------------------------------------|----------------------------------------------------------|----------------|
| Требует полного доступа к галерее?    | Да (NSPhotoLibraryUsageDescription)                | **Нет** — работает с ограниченным доступом (limited)     | ★★★★★ (приватность) |
| Поддержка множественного выбора       | Нет                                                | **Да** — можно выбрать несколько фото/видео              | ★★★★★ |
| Поддержка Live Photos, ProRAW, HEIC   | Частично                                           | Полная поддержка всех современных форматов               | ★★★★★ |
| Съёмка камерой                        | Да                                                 | **Нет** — только выбор из библиотеки                     | — (если нужна камера → UIImagePicker) |
| Запрос разрешения                     | Полный доступ → большой алерт                      | Ограниченный доступ → мягкий алерт + выбор конкретных фото | ★★★★★ |
| Кастомизация интерфейса               | Почти нет                                          | Фильтры по типу (фото/видео/Live), максимум элементов    | ★★★★☆ |
| Производительность и плавность        | Средняя                                            | Значительно лучше (нативный Photos UI)                   | ★★★★★ |
| Совместимость                         | iOS 4+                                             | **iOS 14+**                                              | — |

**Вывод 2026 года**:
- Хотите **выбрать фото/видео** → **всегда** используйте `PHPickerViewController`
- Хотите **съёмку камерой** → всё ещё `UIImagePickerController`
- Делаете новое приложение → **только** PHPicker (если камера не нужна)

### Основные возможности PHPickerViewController

| Свойство / Метод                   | Тип / Значение                                         | Что делает / зачем нужен                                           | Самый частый сценарий            |
| ---------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------ | -------------------------------- |
| `configuration`                    | `PHPickerConfiguration`                                | Основная настройка (фильтры, лимит, режим выбора)                  | Обязательно задавать             |
| `filter`                           | `PHPickerFilter` (.images, .videos, .livePhotos и др.) | Какие типы медиа показывать                                        | `.images` — самый популярный     |
| `selectionLimit`                   | `Int` (0 = без ограничения, 1 = одиночный выбор)       | Максимальное количество выбираемых элементов                       | `0` или `1`                      |
| `preferredAssetRepresentationMode` | `.automatic` / `.current` / `.unmodified`              | В каком формате возвращать ([[HEIC]]/ProRAW или конвертировать)    | `.automatic` — обычно достаточно |
| `selection`                        | `[PHPickerResult]`                                     | Результаты выбора (после завершения)                               | Основной результат               |
| `itemProvider` (в PHPickerResult)  | `NSItemProvider`                                       | Доступ к данным ([[data]], [[UIImage]], [[URL]], LivePhoto и т.д.) | Загрузка контента                |

### Полный современный пример (2026 — [[SwiftUI]] + [[UIKit]] + [[async]]/[[await]])

#### Вариант 1. UIKit (классический делегат)

```swift
import UIKit
import PhotosUI

class ProfileViewController: UIViewController, PHPickerViewControllerDelegate {
    
    private let imageView = UIImageView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // настройка imageView...
    }
    
    @objc private func pickPhoto() {
        var config = PHPickerConfiguration(photoLibrary: .shared())
        
        // Самые частые настройки
        config.filter = .images                     // только фото (можно .videos, .livePhotos)
        config.selectionLimit = 1                   // одиночный выбор (0 = много)
        config.preferredAssetRepresentationMode = .automatic
        
        let picker = PHPickerViewController(configuration: config)
        picker.delegate = self
        
        present(picker, animated: true)
    }
    
    // Делегат — вызывается после выбора
    func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
        picker.dismiss(animated: true)
        
        guard let provider = results.first?.itemProvider,
              provider.canLoadObject(ofClass: UIImage.self) else { return }
        
        provider.loadObject(ofClass: UIImage.self) { [weak self] image, error in
            DispatchQueue.main.async {
                if let image = image as? UIImage {
                    self?.imageView.image = image
                    // можно сохранить или загрузить на сервер
                }
            }
        }
    }
}
```

#### Вариант 2. SwiftUI + PhotosPicker (iOS 16+ — самый чистый способ)

```swift
import SwiftUI
import PhotosUI

struct ProfileView: View {
    
    @State private var selectedItems: [PhotosPickerItem] = []
    @State private var selectedImage: Image?
    
    var body: some View {
        VStack {
            if let selectedImage {
                selectedImage
                    .resizable()
                    .scaledToFit()
                    .frame(height: 300)
                    .clipShape(RoundedRectangle(cornerRadius: 16))
            } else {
                Image(systemName: "photo")
                    .resizable()
                    .scaledToFit()
                    .frame(height: 200)
                    .foregroundStyle(.gray)
            }
            
            PhotosPicker(
                selection: $selectedItems,
                maxSelectionCount: 1,
                matching: .images
            ) {
                Label("Выбрать фото", systemImage: "photo.on.rectangle")
                    .font(.headline)
            }
            .buttonStyle(.borderedProminent)
        }
        .onChange(of: selectedItems) { oldValue, newValue in
            Task {
                if let item = newValue.first,
                   let data = try? await item.loadTransferable(type: Data.self),
                   let uiImage = UIImage(data: data) {
                    selectedImage = Image(uiImage: uiImage)
                }
            }
        }
    }
}
```

### Лучшие практики PHPickerViewController в 2026 году

- **Всегда** задавайте `configuration` — без неё picker покажет всё подряд  
- **Используйте** `filter = .images` — если вам нужны только фото (`.videos`, `.livePhotos`, `.all(of: [.images, .videos])`)  
- **Для одиночного выбора** — `selectionLimit = 1` (по умолчанию 1)  
- **Для приватности** — PHPicker работает даже при **ограниченном доступе** к галерее — пользователь выбирает конкретные фото  
- **Для SwiftUI** — **PhotosPicker** (iOS 16+) — это нативный и самый удобный компонент  
- **Для UIKit** — делегат `PHPickerViewControllerDelegate` + `loadObject(ofClass:)` или `loadTransferable`  
- **Privacy Manifest** — **не требуется** для PHPicker (это UI-компонент, не собирает данные)  
- **Документируйте** — пишите комментарий «PHPickerViewController — современный выбор фото из галереи с поддержкой ограниченного доступа и множественного выбора»

**Короткий итог 2026**:
> PHPickerViewController — это **современный и приватный** контроллер выбора фото/видео из галереи (iOS 14+).  
> В 2026 году:  
> - заменяет `UIImagePickerController` во всех случаях, где **не нужна камера**  
> - поддерживает множественный выбор, Live Photos, ProRAW, ограниченный доступ  
> - не требует полного доступа к галерее — лучший UX и приватность  
> - в SwiftUI — используйте `PhotosPicker`  
> Это **единственный рекомендуемый** способ выбора медиа из библиотеки в новых приложениях.
