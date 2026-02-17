**multipart/form-data** (часто называют просто **form-data** или **multipart**) — это один из самых распространённых способов отправки данных на сервер через [[HTTP]]-запрос, когда нужно передать не только простые текстовые поля, но и **файлы**, **бинарные данные** или смесь разных типов контента.

### Краткое сравнение основных Content-Type для POST-запросов (2025–2026)

| Content-Type                        | Когда использовать                                        | Передача файлов     | Передача нескольких полей | Кодирование | Размер payload  | Самый частый сценарий в iOS/Swift      |
| ----------------------------------- | --------------------------------------------------------- | ------------------- | ------------------------- | ----------- | --------------- | -------------------------------------- |
| `application/x-www-form-urlencoded` | Простые формы (логин, поиск, фильтры)                     | Нет                 | Да                        | URL-encoded | Маленький       | Логин, поиск, простые API              |
| `application/json`                  | Современные [[REST]]/[[GraphQL]] [[API]], [[JSON]]-данные | Нет (только base64) | Да                        | JSON        | Средний–большой | Почти все новые API                    |
| **multipart/form-data**             | Загрузка файлов + текстовые поля одновременно             | **Да**              | **Да**                    | multipart   | Большой         | Фото профиля, аватар, документы, видео |
| `application/octet-stream`          | Передача одного сырого файла без дополнительных полей     | Да                  | Нет                       | бинарный    | Большой         | Прямой upload файла без метаданных     |

### Когда именно нужен multipart/form-data

- Пользователь загружает **фото профиля / аватар**  
- Прикрепление **документов**, **сканов**, **видео**, **аудио**  
- Форма с несколькими полями + хотя бы один файл  
- Множественная загрузка файлов (multi-file upload)  
- Нужно передать **имя файла**, **mime-тип**, **дополнительные метаданные** вместе с файлом

### Как правильно отправлять multipart/form-data в Swift (2026 актуальные способы)

#### Вариант 1 — Самый популярный и рекомендуемый ([[URLSession]] + вручную собираем body)

```swift
func uploadAvatar(image: UIImage, userId: String) async throws -> Data {
    let boundary = "Boundary-\(UUID().uuidString)"
    var request = URLRequest(url: URL(string: "https://api.example.com/upload")!)
    request.httpMethod = "POST"
    request.setValue("multipart/form-data; boundary=\(boundary)", forHTTPHeaderField: "Content-Type")
    
    var body = Data()
    
    // Текстовое поле
    body.append("--\(boundary)\r\n".data(using: .utf8)!)
    body.append("Content-Disposition: form-data; name=\"user_id\"\r\n\r\n".data(using: .utf8)!)
    body.append("\(userId)\r\n".data(using: .utf8)!)
    
    // Файл
    guard let imageData = image.jpegData(compressionQuality: 0.8) else {
        throw UploadError.invalidImage
    }
    
    body.append("--\(boundary)\r\n".data(using: .utf8)!)
    body.append("Content-Disposition: form-data; name=\"avatar\"; filename=\"avatar.jpg\"\r\n".data(using: .utf8)!)
    body.append("Content-Type: image/jpeg\r\n\r\n".data(using: .utf8)!)
    body.append(imageData)
    body.append("\r\n".data(using: .utf8)!)
    
    // Завершение
    body.append("--\(boundary)--\r\n".data(using: .utf8)!)
    
    request.httpBody = body
    
    let (data, _) = try await URLSession.shared.data(for: request)
    return data
}
```

#### Вариант 2 — Самый удобный и рекомендуемый в 2026 — **[[Alamofire]]** (multipartFormData)

```swift
import Alamofire

func uploadAvatarAlamofire(image: UIImage, userId: String) async throws {
    let url = "https://api.example.com/upload"
    
    let formData = MultipartFormData()
    formData.append(userId.data(using: .utf8)!, withName: "user_id")
    
    if let imageData = image.jpegData(compressionQuality: 0.8) {
        formData.append(imageData, withName: "avatar", fileName: "avatar.jpg", mimeType: "image/jpeg")
    }
    
    let request = AF.upload(multipartFormData: formData, to: url)
    
    let response = await request.serializingData().value
    
    // обработка response
}
```

**Alamofire** в 2026 году остаётся **самым удобным** способом для multipart-запросов.

#### Вариант 3 — [[SwiftUI]] + PhotosPicker + multipart (очень популярный сценарий 2026)

```swift
@State private var selectedItem: PhotosPickerItem?
@State private var imageData: Data?

var body: some View {
    PhotosPicker("Выбрать фото", selection: $selectedItem, matching: .images)
        .onChange(of: selectedItem) { newItem in
            Task {
                if let data = try? await newItem?.loadTransferable(type: Data.self) {
                    imageData = data
                    try await uploadAvatar(data: data)
                }
            }
        }
}

private func uploadAvatar(data: Data) async throws {
    // тот же multipart-код через Alamofire или URLSession
}
```

### Лучшие практики multipart/form-data в Swift 2026

- **Alamofire** — самый читаемый и надёжный способ (рекомендуется для большинства проектов)  
- **boundary** — генерируй уникальный [[UUID]]-based boundary  
- **Content-Disposition** — всегда указывай `name` и `filename` для файлов  
- **Content-Type** — корректный mime-тип (`image/jpeg`, `application/pdf`, `image/png` и т.д.)  
- **[[async]]/[[await]]** — всегда оборачивай в `async throws`  
- **Swift 6 strict concurrency** — все сетевые операции в [[Task]] или [[actor]]  
- **Размер файла** — проверяй перед отправкой (обычно лимит 10–50 МБ на сервере)  
- **Прогресс** — используй `uploadProgress` в [[Alamofire]] или `URLSessionTaskDelegate`  
- **Тестирование** — мокай через `URLProtocol` или `Alamofire` stub  

**Короткий девиз 2026**:
> «multipart/form-data — это когда нужно отправить файл + текстовые поля одновременно.  
> В 2026 году самый удобный и надёжный способ — **Alamofire multipartFormData**.  
> Ручной URLSession — только если очень нужен минимализм или нет зависимости от Alamofire.»
