Чтение и запись файлов в [[iOS]] можно реализовать с помощью классов [[FileManager]] и [[Data]]. Вот примеры того, как это можно сделать:

1. **Чтение файла**:
```swift
if let filePath = Bundle.main.path(forResource: "example", ofType: "txt") {
    do {
        let content = try String(contentsOfFile: filePath, encoding: .utf8)
        print(content)
    } catch {
        print("Ошибка чтения файла: \(error.localizedDescription)")
    }
} else {
    print("Файл не найден")
}

```
**Запись файла**:
```swift
let text = "Пример текста для записи в файл"
let fileName = "example.txt"

if let documentsDirectory = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first {
    let fileURL = documentsDirectory.appendingPathComponent(fileName)
    
    do {
        try text.write(to: fileURL, atomically: false, encoding: .utf8)
        print("Файл успешно записан: \(fileURL.absoluteString)")
    } catch {
        print("Ошибка записи файла: \(error.localizedDescription)")
    }
}

```
Обратите внимание, что во втором примере файл будет записан в директорию `Documents` вашего приложения. Также обратите внимание на обработку ошибок при чтении и записи файлов, чтобы ваше приложение корректно обрабатывало возможные проблемы во время выполнения операций ввода-вывода.