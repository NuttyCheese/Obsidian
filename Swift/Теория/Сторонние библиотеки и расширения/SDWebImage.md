#third-party_library #Swift 
## 📘 Определение

**SDWebImage** — популярная библиотека для [[iOS]], позволяющая **асинхронно загружать изображения из сети** с кэшированием в памяти и на диске.  
Обеспечивает **автоматическое управление кэшем, отображение placeholder и обработку ошибок загрузки**.  
Относится к **[[UIKit]] → [[UIImageView]] / Networking**.

---

## 🔹 Примеры кода

### 1. Простейшая загрузка изображения в UIImageView

```swift
import SDWebImage
import UIKit

let imageView = UIImageView()
let url = URL(string: "https://example.com/image.png")
imageView.sd_setImage(with: url)
```

---

### 2. Загрузка с placeholder

```swift
let placeholder = UIImage(named: "placeholder")
imageView.sd_setImage(with: url, placeholderImage: placeholder)
```

---

### 3. Обработка завершения загрузки

```swift
imageView.sd_setImage(with: url, completed: { image, error, cacheType, url in
    if let error = error {
        print("Ошибка загрузки:", error)
    } else {
        print("Изображение загружено, cacheType:", cacheType)
    }
})
```

---

### 4. Очистка кэша

```swift
SDImageCache.shared.clearMemory() // очистка памяти
SDImageCache.shared.clearDisk(onCompletion: {
    print("Кэш на диске очищен")
})
```

---

### 5. Использование с [[SwiftUI]] через UIViewRepresentable

```swift
import SwiftUI
import SDWebImageSwiftUI

struct RemoteImageView: View {
    let url: URL?
    
    var body: some View {
        WebImage(url: url)
            .resizable()
            .placeholder(Image(systemName: "photo"))
            .scaledToFit()
            .frame(width: 200, height: 200)
    }
}
```
