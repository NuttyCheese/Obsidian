## 📘 Определение

**Sandbox** — изолированная среда, в которой выполняется приложение на [[iOS]].  
Ограничивает **доступ к файлам, настройкам и системным ресурсам**, обеспечивая **безопасность и защиту данных**.  
Каждое приложение имеет свой **песочницу (sandbox)** с отдельной директорией для документов, кэша и временных файлов.  
Относится к **iOS → Security / File System**.

---

## 🔹 Примеры кода

### 1. Получение пути к директории Documents

```swift
let documentsURL = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
print("Documents path:", documentsURL.path)
```

---

### 2. Запись файла в sandbox

```swift
let fileURL = documentsURL.appendingPathComponent("example.txt")
let text = "Hello Sandbox"
try? text.write(to: fileURL, atomically: true, encoding: .utf8)
```

---

### 3. Чтение файла из sandbox

```swift
if let content = try? String(contentsOf: fileURL) {
    print("Содержимое файла:", content)
}
```

---

### 4. Получение пути к временной директории

```swift
let tempURL = FileManager.default.temporaryDirectory
print("Temp path:", tempURL.path)
```

---

### 5. Очистка временной директории

```swift
let tempDir = FileManager.default.temporaryDirectory
let tempFiles = try? FileManager.default.contentsOfDirectory(atPath: tempDir.path)

tempFiles?.forEach { file in
    let filePath = tempDir.appendingPathComponent(file).path
    try? FileManager.default.removeItem(atPath: filePath)
}
```
