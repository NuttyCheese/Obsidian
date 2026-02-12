#third-party_library 
## 📘 Определение

**Kingfisher** — это [[Swift]]-библиотека для **асинхронной загрузки и кэширования изображений** в [[iOS]]-приложениях.  
Позволяет легко загружать изображения из сети, отображать их в [[UIImageView]], кэшировать локально и обрабатывать с помощью **фильтров и трансформаций**.  
Относится к **UI / Network layer**.

---

## 🔹 Примеры кода

### 1. Простая загрузка изображения в [[UIImageView]]

```swift
import Kingfisher
import UIKit

let imageView = UIImageView()
let url = URL(string: "https://example.com/image.jpg")
imageView.kf.setImage(with: url)
```

---

### 2. Загрузка с плейсхолдером

```swift
let placeholder = UIImage(systemName: "photo")
imageView.kf.setImage(with: url, placeholder: placeholder)
```

---

### 3. Загрузка с обработкой ошибок

```swift
imageView.kf.setImage(with: url) { result in
    switch result {
    case .success(let value):
        print("Изображение загружено:", value.source.url?.absoluteString ?? "")
    case .failure(let error):
        print("Ошибка загрузки:", error)
    }
}
```

---

### 4. Применение фильтров к изображению

```swift
import Kingfisher

let processor = RoundCornerImageProcessor(cornerRadius: 20)
imageView.kf.setImage(with: url, options: [.processor(processor)])
```

---

### 5. Предварительная загрузка изображений (prefetch)

```swift
let urls = [
    URL(string: "https://example.com/1.jpg")!,
    URL(string: "https://example.com/2.jpg")!
]

let prefetcher = ImagePrefetcher(urls: urls)
prefetcher.start()
```

---

### 6. Кэширование и очистка кэша

```swift
let cache = ImageCache.default

// Сохраняем изображение вручную
cache.store(UIImage(named: "example")!, forKey: "localImage")

// Получаем из кэша
cache.retrieveImage(forKey: "localImage") { result in
    switch result {
    case .success(let value):
        print("Изображение из кэша:", value.image ?? "нет")
    case .failure(let error):
        print(error)
    }
}

// Очистка кэша
cache.clearMemoryCache()
cache.clearDiskCache()
```
