**Callback** (обратный вызов) — это **функция** (или **замыкание** — closure), которую ты передаёшь как аргумент другой функции, и она будет вызвана **позже**, когда та функция закончит свою работу или захочет сообщить результат.

Это классический паттерн асинхронного программирования до появления `async/await` в Swift 5.5 (2021). В 2026 году callback всё ещё жив, но его роль сильно уменьшилась — теперь почти везде предпочтительнее `async/await` + `Task`.

### 1. Зачем вообще нужны callback’ы (реальные причины 2026)

| Ситуация                                      | Почему callback был нужен раньше                     | Как сейчас решается лучше (2026)                     |
|-----------------------------------------------|-------------------------------------------------------|------------------------------------------------------|
| Сетевой запрос (API, загрузка фото)           | `URLSession.dataTask` требует completion handler      | `async let (data, _) = URLSession.shared.data(from:)` |
| Анимация с завершающим действием              | `UIView.animate { ... } completion: { ... }`          | `withAnimation { ... }` + `Task { await ... }`       |
| Таймер / отложенное действие                  | `Timer.scheduledTimer` + selector или closure         | `Task { try await Task.sleep(...) }`                 |
| Работа с файлами / Core Data / Realm          | Completion-блоки в старых API                         | `async` методы в современных фреймворках             |
| Обработка нескольких асинхронных операций     | Callback hell (вложенные замыкания)                   | `async let`, `TaskGroup`, `actor`, `withThrowingTaskGroup` |
| Совместимость со старым кодом                 | UIKit, Objective-C, сторонние библиотеки до 2021–2023 | Миграция на `async` + `@available` / `#if`           |

### 2. Два главных типа callback’ов в Swift

| Тип                     | Когда используется                                     | Как пишется в 2026 году (рекомендуется)              | Пример |
|-------------------------|--------------------------------------------------------|------------------------------------------------------|--------|
| **Non-escaping closure** | Вызывается **внутри** функции, не сохраняется         | `@escaping` **не пишется**                           | `map`, `filter`, `sorted` |
| **Escaping closure**     | Может быть сохранён и вызван **позже** (асинхронно)   | `@escaping` **обязательно**                          | `completion` в `URLSession`, `animate` |

**Важно**: с Swift 5.7+ компилятор сам подсказывает, нужен ли `@escaping`.  
Если closure сохраняется (например, в свойство, очередь, замыкание) → нужен `@escaping`.

### 3. Самый популярный и современный паттерн 2026 года

#### Вариант 1: Классический escaping callback (ещё часто встречается в UIKit)

```swift
func loadImage(from url: URL, completion: @escaping (Result<UIImage, Error>) -> Void) {
    URLSession.shared.dataTask(with: url) { data, response, error in
        if let error {
            completion(.failure(error))
            return
        }
        
        guard let data, let image = UIImage(data: data) else {
            completion(.failure(NSError(domain: "ImageError", code: -1)))
            return
        }
        
        completion(.success(image))
    }.resume()
}

// Использование
loadImage(from: imageURL) { result in
    switch result {
    case .success(let image):
        DispatchQueue.main.async {
            self.imageView.image = image
        }
    case .failure(let error):
        print("Ошибка загрузки: \(error)")
    }
}
```

#### Вариант 2: Callback → async/await (миграция 2026)

```swift
// Современный вариант той же функции
func loadImage(from url: URL) async throws -> UIImage {
    let (data, _) = try await URLSession.shared.data(from: url)
    
    guard let image = UIImage(data: data) else {
        throw ImageError.invalidData
    }
    
    return image
}

// Использование — чисто и красиво
Task {
    do {
        let image = try await loadImage(from: imageURL)
        await MainActor.run {
            self.imageView.image = image
        }
    } catch {
        print("Ошибка: \(error)")
    }
}
```

### 4. Как избежать callback hell (самая большая проблема старого кода)

Старый стиль (callback hell):

```swift
step1 { result1 in
    step2(result1) { result2 in
        step3(result2) { result3 in
            step4(result3) { result4 in
                // ... ад вложенности
            }
        }
    }
}
```

Современный стиль 2026 (async/await + Structured Concurrency):

```swift
Task {
    do {
        let result1 = try await step1()
        let result2 = try await step2(result1)
        let result3 = try await step3(result2)
        let result4 = try await step4(result3)
        // чисто и линейно
    } catch {
        print("Ошибка: \(error)")
    }
}
```

Или ещё лучше — с `async let` (параллельное выполнение):

```swift
async let r1 = step1()
async let r2 = step2()
async let r3 = step3()

let (result1, result2, result3) = try await (r1, r2, r3)
```

### 5. Лучшие практики callback в Swift 2026

- **Мигрируй на async/await** — почти все новые API (URLSession, FileManager, Core Data, CloudKit) имеют `async` версии  
- **Если API только с callback** — оборачивай в `async` функцию с `withCheckedThrowingContinuation`  
- **Всегда используй `[weak self]` в escaping closure** — иначе retain cycle  
- **Передавай результат через `Result<Type, Error>`** — это стандарт  
- **Для UI** — всегда диспатчи на главный поток: `DispatchQueue.main.async` или `await MainActor.run`  
- **Swift 6 strict concurrency** — escaping closure должен захватывать `self` явно (`[weak self]` или `[self]`)  
- **Документируйте** — пиши комментарий «@escaping completion — вызывается после загрузки данных»

**Короткий девиз 2026**:
> Callback — это когда функция говорит: «когда я закончу — позвоню тебе».  
> В 2026 году callback’и почти везде заменены на `async/await` — код стал линейным, читаемым и безопасным.  
> Если видишь старый callback — оборачивай его в `async` функцию и живи счастливо.

Удачи с чистым асинхронным кодом без callback hell в твоём приложении! 🚀