**`Data`** в Swift — это **универсальный контейнер для байтовых данных** (сырых байтов), который используется практически во всех сценариях, где нужно работать с двоичным контентом: изображения, аудио, видео, JSON, plist, сетевые ответы, шифрование, файлы, сертификаты, protobuf, ASN.1 и т.д.

Это **один из самых часто используемых типов** в реальных iOS/macOS-приложениях.

### 1. Ключевые характеристики Data (2026 актуально)

| Характеристика                        | Значение / Особенность                                      | Важные детали 2026 |
|---------------------------------------|-------------------------------------------------------------|---------------------|
| Тип данных                            | struct (value type)                                         | Copy-on-Write (COW) |
| Внутреннее представление              | Contiguous блок памяти (UInt8)                              | Эффективно для больших объёмов |
| Основной способ создания              | `Data(contentsOf:)`, `data(using:)`, `Data(bytes:)`         | — |
| Изменяемость                          | `var` — mutable, `let` — immutable                          | Mutable операции — append, replaceSubrange |
| Thread-safety                         | Не является thread-safe (требует синхронизации)             | В Swift 6 строже проверяется |
| Основные сценарии использования       | Сеть, файлы, шифрование, изображения, аудио, protobuf       | Почти все асинхронные API |

### 2. Самые популярные способы создания Data (2026)

```swift
// 1. Из строки (самый частый)
let text = "Hello, world!"
let utf8Data = text.data(using: .utf8)          // Data?
let utf16Data = text.data(using: .utf16)        // редко

// 2. Из массива байтов
let bytes: [UInt8] = [0x48, 0x65, 0x6C, 0x6C, 0x6F] // "Hello"
let dataFromBytes = Data(bytes)

// 3. Из файла (очень частый)
if let fileData = try? Data(contentsOf: url) {
    // ...
}

// 4. Пустой mutable Data
var mutableData = Data()                        // 0 байт
mutableData.append("More text".data(using: .utf8)!)

// 5. Из UIImage / AVAsset / Data(contentsOf: URL)
let imageData = UIImage(systemName: "star")?.pngData()
let videoData = try? Data(contentsOf: videoURL)
```

### 3. Самые полезные операции с Data

```swift
var data = Data()

// Добавление
data.append("Hello".data(using: .utf8)!)
data.append([0x20, 0x77, 0x6F, 0x72, 0x6C, 0x64])  // " world"

// Чтение как массив байтов
let bytes = [UInt8](data)                           // [72, 101, 108, 108, 111, 32, 119, ...]

// Преобразование в строку
let string = String(data: data, encoding: .utf8)

// Base64 (очень частый)
let base64 = data.base64EncodedString()             // "SGVsbG8gd29ybGQ="

// JSON / Codable
let jsonData = try JSONEncoder().encode(user)
let decoded = try JSONDecoder().decode(User.self, from: jsonData)

// Диапазон (subdata)
let prefix = data.prefix(5)                         // первые 5 байт
let subdata = data.subdata(in: 0..<10)

// Замена / вставка
data.replaceSubrange(0..<5, with: "Hi".data(using: .utf8)!)

// Подсчёт байтов
print(data.count)                                   // в байтах, не символах!
```

### 4. Самый популярный паттерн 2026 года (реальная жизнь)

```swift
@MainActor
class ProfileViewModel: ObservableObject {
    
    @Published var avatar: UIImage?
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    func loadAvatar(from url: URL) async {
        isLoading = true
        errorMessage = nil
        
        do {
            let (data, _) = try await URLSession.shared.data(from: url)
            
            guard let image = UIImage(data: data) else {
                throw ImageError.invalidData
            }
            
            await MainActor.run {
                self.avatar = image
            }
        } catch {
            await MainActor.run {
                self.errorMessage = error.localizedDescription
            }
        }
        
        await MainActor.run {
            self.isLoading = false
        }
    }
}
```

### 5. Лучшие практики Data в Swift 2026

- **Всегда** используй `try await URLSession.shared.data(from:)` вместо старого `dataTask`  
- **Проверяй encoding** при преобразовании в строку: `.utf8` — почти всегда  
- **Не храни большие Data в памяти** — используй `Data(contentsOf:)` с `options: .mappedIfSafe` для файлов  
- **Для JSON** — используй `Codable` + `JSONEncoder`/`Decoder` вместо ручного парсинга  
- **Для Base64** — `base64EncodedString(options:)` и `init(base64Encoded:)`  
- **Для больших файлов** — используй `FileHandle` или `InputStream` вместо полного `Data(contentsOf:)`  
- **Swift 6 strict concurrency** — `Data` полностью `Sendable` и безопасен  
- **Документируйте** — пиши комментарий «Data — содержимое аватарки в формате PNG»

**Короткий девиз 2026**:
> `Data` — это **универсальный контейнер байтов** для всего двоичного: изображений, JSON, файлов, сетевых ответов, шифрования.  
> В 2026 году:  
> - используй `async/await` + `URLSession.data(from:)`  
> - работай через `Codable` для структур  
> - проверяй `count` в байтах, а не символах  
> - избегай больших `Data(contentsOf:)` в памяти  

Удачи с эффективной работой с байтами в твоём приложении! 📦