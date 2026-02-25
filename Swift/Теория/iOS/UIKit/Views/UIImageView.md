**`UIImageView`** — это подкласс **[[UIView]]**, предназначенный исключительно для **отображения одного или нескольких изображений** ([[UIImage]]) в [[iOS]]-приложениях.

Это один из самых часто используемых элементов интерфейса в [[UIKit]]. В 2026 году он остаётся актуальным, несмотря на появление `Image` в [[SwiftUI]], потому что:
- даёт полный контроль над производительностью и поведением,
- поддерживает анимацию, gesture recognizers, CALayer-эффекты,
- идеально работает в смешанных UIKit + SwiftUI проектах через [[UIViewRepresentable]].

### Основные характеристики UIImageView (актуально на 2026 год)

| Свойство / Возможность     | Значение по умолчанию     | Что делает / зачем нужен                         | Самый частый сценарий                                    |
| -------------------------- | ------------------------- | ------------------------------------------------ | -------------------------------------------------------- |
| `image`                    | [[nil]]                   | Основное изображение (один кадр)                 | Отображение фото, иконки                                 |
| `highlightedImage`         | `nil`                     | Изображение в выделенном состоянии (при нажатии) | Кнопки, иконки в таббаре                                 |
| `animationImages`          | `nil`                     | Массив изображений для анимации                  | Лоадеры, индикаторы, GIF-подобные эффекты                |
| `animationDuration`        | 0.033 сек × кол-во кадров | Длительность полного цикла анимации              | Настройка скорости                                       |
| `animationRepeatCount`     | 0 (бесконечно)            | Количество повторов анимации                     | Одноразовые эффекты                                      |
| `contentMode`              | `.scaleToFill`            | Как изображение масштабируется и позиционируется | `.scaleAspectFit`, `.scaleAspectFill` — самые популярные |
| `isUserInteractionEnabled` | `false`                   | Разрешает ли вью получать касания                | Включать для тапов, long press                           |
| `clipsToBounds`            | `false`                   | Обрезает ли содержимое по границам view          | Почти всегда `true` при cornerRadius                     |
| `tintColor`                | наследуется от superview  | Цвет для template-изображений                    | Иконки SF Symbols                                        |

### Основные contentMode (самые используемые в 2026)

| contentMode                  | Описание визуально                                      | Когда использовать |
|------------------------------|----------------------------------------------------------|---------------------|
| `.scaleToFill`               | Растягивает изображение, заполняя весь view (может искажать) | Редко |
| `.scaleAspectFit`            | Вписывает изображение, сохраняя пропорции (может быть пустое пространство) | Фото профиля, логотипы, контент |
| `.scaleAspectFill`           | Заполняет весь view, сохраняя пропорции (обрезает края) | Фоны, аватарки в круге |
| `.center`                    | Центрирует без масштабирования                           | Иконки фиксированного размера |
| `.top`, `.bottom`, `.left`, `.right` и комбинации | Привязка к краю без масштабирования                      | Специфические дизайны |

### Самые популярные и рекомендуемые паттерны 2026 года

#### 1. Базовое использование (самый частый)

```swift
let imageView = UIImageView(image: UIImage(named: "avatar"))
imageView.contentMode = .scaleAspectFill
imageView.clipsToBounds = true
imageView.layer.cornerRadius = 16
view.addSubview(imageView)
```

#### 2. SF Symbols с tintColor (очень популярно)

```swift
let iconView = UIImageView()
iconView.image = UIImage(systemName: "heart.fill")?
    .withRenderingMode(.alwaysTemplate)
iconView.tintColor = .systemRed
iconView.contentMode = .scaleAspectFit
```

#### 3. Анимация (лоадер, индикатор)

```swift
let loadingImages = (1...12).compactMap { UIImage(named: "loading\($0)") }
imageView.animationImages = loadingImages
imageView.animationDuration = 1.0
imageView.animationRepeatCount = 0
imageView.startAnimating()
```

#### 4. UIImageView с распознаванием жестов

```swift
imageView.isUserInteractionEnabled = true

let tap = UITapGestureRecognizer(target: self, action: #selector(imageTapped))
imageView.addGestureRecognizer(tap)

@objc private func imageTapped() {
    print("Картинка нажата")
    // открываем полноэкранный просмотр
}
```

#### 5. Кэширование и асинхронная загрузка (рекомендуемый способ 2026)

```swift
extension UIImageView {
    func loadImage(from url: URL, placeholder: UIImage? = nil) {
        image = placeholder
        
        URLSession.shared.dataTask(with: url) { [weak self] data, _, error in
            guard let data, error == nil,
                  let image = UIImage(data: data) else { return }
            
            DispatchQueue.main.async {
                self?.image = image
            }
        }.resume()
    }
}

// Использование
imageView.loadImage(from: URL(string: "https://...")!, placeholder: UIImage(systemName: "photo"))
```

### Лучшие практики UIImageView в Swift 2026

- **Всегда** задавайте `contentMode` — по умолчанию `.scaleToFill` часто искажает изображение  
- **Для круглых аватарок** — `clipsToBounds = true` + `layer.cornerRadius = bounds.height / 2`  
- **Для иконок** — используйте SF Symbols + `.alwaysTemplate` + `tintColor`  
- **Для анимаций** — `animationImages` — самый лёгкий и производительный способ  
- **Для сетевых изображений** — используйте кэширование (Kingfisher, SDWebImage, Nuke, или свой) — ручная загрузка через URLSession устарела  
- **Для [[SwiftUI]]** — используйте `Image` / `AsyncImage` — `UIImageView` нужен только в смешанных проектах через `UIViewRepresentable`  
- **Для производительности** — избегайте большого количества анимированных UIImageView одновременно  
- **Документируйте** — пишите комментарий «UIImageView — аватарка пользователя с scaleAspectFill и скруглением»

**Короткий итог 2026**:
> UIImageView — это **специализированный UIView** для отображения изображений (`UIImage`).  
> В 2026 году:  
> - ключевые свойства — `image`, `contentMode`, `clipsToBounds`, `animationImages`  
> - для иконок — SF Symbols + `.alwaysTemplate`  
> - для сетевых изображений — используйте кэширующие библиотеки  
> - для жестов — включайте `isUserInteractionEnabled` и добавляйте recognizers  
> Это **самый простой** и **самый часто встречающийся** способ показать картинку в UIKit-приложении.
