**`CGImage`** — это структура (C-структура) из **Core Graphics** (Quartz), представляющая **растровое изображение** в памяти — набор пикселей с цветами, альфа-каналом, размером, цветовым пространством и другими метаданными.

Это **низкоуровневый** объект, который используется практически во всех случаях, когда нужно работать с изображениями на уровне пикселей, Core Graphics, Core Animation, Metal, AVFoundation, Image I/O и т.д.

### Ключевые характеристики CGImage (актуально на 2026 год)

| Характеристика                     | Описание                                                                 | Важные замечания 2026 |
|------------------------------------|--------------------------------------------------------------------------|------------------------|
| Тип                                | `CGImage` (C-структура, не объект)                                       | Не является `NSObject` / `AnyObject` |
| Компоненты                         | Пиксели (обычно 4 байта на пиксель для RGBA)                             | Поддерживает разные форматы (RGB, grayscale, CMYK и др.) |
| Color Space                        | `CGColorSpace` (sRGB, P3, DeviceRGB, Display P3 и т.д.)                  | Чаще всего sRGB или Display P3 |
| Bits per component                 | Обычно 8 (24/32 бита на пиксель)                                         | Может быть 16 бит (HDR) |
| Alpha info                         | `CGImageAlphaInfo` (premultiplied, last, none и др.)                     | Важно для прозрачности |
| Создание                           | Через `UIImage.cgImage`, `CGImageSourceCreateImageAtIndex`, `CGBitmapContextCreateImage` и т.д. | Почти всегда через `UIImage` |
| Swift-обёртка                      | `CGImage` → `UIImage` / `Image` (SwiftUI)                                | Рекомендуется работать через Swift-типы |

### Самые частые способы получения CGImage в 2026 году

#### 1. Из UIImage (самый популярный и рекомендуемый)

```swift
let uiImage = UIImage(named: "example") ?? UIImage(systemName: "photo")!
let cgImage = uiImage.cgImage

let layer = CALayer()
layer.contents = cgImage  // ← здесь нужен CGImage
```

#### 2. Создание CGImage из данных (через Data / CGImageSource)

```swift
guard let data = UIImage(named: "photo")?.pngData(),
      let source = CGImageSourceCreateWithData(data as CFData, nil),
      let cgImage = CGImageSourceCreateImageAtIndex(source, 0, nil) else {
    return
}

// Теперь cgImage готов к использованию
```

#### 3. Создание CGImage из контекста (рисование в CGBitmapContext)

```swift
let width = 300
let height = 200
let colorSpace = CGColorSpaceCreateDeviceRGB()
let bitmapInfo = CGBitmapInfo.byteOrder32Little.union(.alphaInfoMask)

guard let context = CGBitmapContextCreate(nil,
                                         width,
                                         height,
                                         8,
                                         width * 4,
                                         colorSpace,
                                         bitmapInfo.rawValue) else { return }

context.setFillColor(UIColor.systemBlue.cgColor)
context.fill(CGRect(x: 0, y: 0, width: width, height: height))

let cgImage = CGBitmapContextCreateImage(context)
```

#### 4. CGImage → UIImage (обратное преобразование)

```swift
let uiImage = UIImage(cgImage: cgImage)
let swiftUIImage = Image(uiImage: uiImage)
```

### Самые популярные сценарии использования CGImage в 2026

1. **CALayer.contents**  
   ```swift
   layer.contents = image.cgImage
   layer.contentsGravity = .resizeAspect
   ```

2. **Core Graphics рисование**  
   ```swift
   context.draw(cgImage, in: rect)
   ```

3. **Metal / Core Image фильтры**  
   ```swift
   let ciImage = CIImage(cgImage: cgImage)
   let filtered = ciImage.applyingFilter("CIGaussianBlur", parameters: [kCIInputRadiusKey: 10])
   ```

4. **Создание текстур для Metal / SceneKit**  
   ```swift
   let texture = try! MTLTexture(from: cgImage, device: device)
   ```

5. **Обработка пикселей** (через `CGBitmapContext`)  
   ```swift
   let data = CGBitmapContextGetData(context)
   let pixelData = data!.assumingMemoryBound(to: UInt8.self)
   // прямой доступ к байтам пикселей
   ```

### Лучшие практики работы с CGImage в Swift 2026

- **Всегда** получайте CGImage **через UIImage** (`uiImage.cgImage`) — это самый безопасный и читаемый способ  
- **Никогда** не создавайте CGImage вручную, если можно обойтись `UIImage` / `CIImage`  
- **Для SwiftUI** — предпочитайте `Image` — `CGImage` нужен только при работе с CALayer или Core Graphics  
- **Для Core Graphics** — всегда проверяйте `CGBitmapContextCreate` на `nil`  
- **Для производительности** — используйте `CGBitmapContext` с правильным `bitmapInfo` и `colorSpace`  
- **Swift 6 strict concurrency** — `CGImage` полностью безопасен (`Sendable`), но контекст и рисование — только на главном потоке  
- **Документируйте** — пишите комментарий «CGImage — растровое изображение для CALayer.contents (из UIImage)»

**Короткий итог 2026**:
> `CGImage` — это **низкоуровневое растровое изображение** для Core Graphics, Core Animation и Metal.  
> В 2026 году:  
> - получайте его **всегда** через `UIImage.cgImage`  
> - используйте для `CALayer.contents`, `CGContext.draw`, Metal-текстур  
> - не создавайте вручную, если не работаете с сырыми пикселями  
> Это **мост** между высокоуровневым UIImage и низкоуровневым рисованием/текстурами  

Удачи с эффективной и безопасной работой с изображениями на уровне пикселей в твоём проекте! 🖼️