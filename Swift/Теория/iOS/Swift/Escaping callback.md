**`@escaping`** — это атрибут в Swift, который указывает, что **замыкание (closure)** может быть **вызвано после того, как функция, в которую оно передано, уже завершила выполнение**.

> Проще говоря: `@escaping` = «это замыкание может жить дольше, чем сама функция».

### 1. Почему нужен @escaping (основная причина)

По умолчанию замыкания в Swift — **non-escaping** (невыходящие за пределы функции):

- Вызываются **синхронно** внутри тела функции
- Компилятор знает, что замыкание не будет сохранено и вызвано позже
- Можно безопасно захватывать `self` сильно (без `weak`/`unowned`)

Если замыкание:

- сохраняется в свойство
- передаётся в другой поток (GCD, OperationQueue)
- используется в асинхронной операции (URLSession, Timer, animation completion)
- возвращается из функции

— тогда оно **escaping** и **обязательно** требует `@escaping`.

Без `@escaping` компилятор выдаст ошибку:

```swift
func asyncTask(completion: () -> Void) {  // Ошибка!
    DispatchQueue.global().async {
        completion()  // ← completion может быть вызвано после выхода из функции
    }
}
```

Правильно:

```swift
func asyncTask(completion: @escaping () -> Void) {
    DispatchQueue.global().async {
        completion()
    }
}
```

### 2. Когда @escaping обязателен (топ-кейсы 2026)

| Сценарий                                      | Почему escaping?                                          | Пример API / паттерн |
|-----------------------------------------------|-----------------------------------------------------------|----------------------|
| Асинхронные операции (GCD, async/await обёртки) | Вызов после выхода из функции                             | `DispatchQueue.async`, `URLSession.dataTask` |
| Completion handler анимаций                   | `completion` вызывается после окончания анимации          | `UIView.animate`, `CAAnimation` |
| Таймеры и отложенные действия                 | Таймер может тикать после выхода из функции               | `Timer.scheduledTimer` |
| Сохранение замыканий в массиве/словаре        | Замыкание живёт дольше функции                            | `completionHandlers.append(completion)` |
| NotificationCenter, KVO, delegates            | Обработчик вызывается позже                               | `addObserver`, `observe` |
| Старые UIKit/Foundation API                   | Почти все completion-based API до async/await             | `CLLocationManager`, `PHPhotoLibrary` |
| Кастомные асинхронные API                     | Когда пишешь свою функцию с отложенным вызовом            | `withCheckedContinuation` обёртки |

### 3. Самый важный паттерн 2026 года: escaping + [weak self]

```swift
class UserProfileViewModel {
    var profileImage: UIImage?
    
    func loadProfileImage(from url: URL, completion: @escaping (Result<UIImage, Error>) -> Void) {
        URLSession.shared.dataTask(with: url) { [weak self] data, response, error in
            guard let self else {
                completion(.failure(CancellationError()))
                return
            }
            
            if let error {
                completion(.failure(error))
                return
            }
            
            guard let data, let image = UIImage(data: data) else {
                completion(.failure(ImageProcessingError.invalidData))
                return
            }
            
            DispatchQueue.main.async {
                self.profileImage = image
                completion(.success(image))
            }
        }.resume()
    }
}
```

**Почему именно так**:
- `[weak self]` → нет retain cycle (ViewModel не удерживает URLSession task)
- `guard let self else { ... }` → безопасный доступ к self после асинхронного вызова
- `DispatchQueue.main.async` → обновление UI на главном потоке

### 4. Полный список самых частых escaping-замыканий в UIKit/Foundation (2026)

| API / Метод                                   | Escaping closure? | Что захватывает? | Рекомендация захвата self |
|-----------------------------------------------|-------------------|------------------|---------------------------|
| `URLSession.dataTask`                         | Да                | data, response, error | `[weak self]` |
| `UIView.animate` completion                   | Да                | —                | `[weak self]` |
| `Timer.scheduledTimer`                        | Да                | —                | `[weak self]` |
| `NotificationCenter.addObserver`              | Да                | —                | `[weak self]` или `object: nil` |
| `CLLocationManager.requestLocation`           | Да                | —                | `[weak self]` |
| `PHPhotoLibrary.requestAuthorization`         | Да                | —                | `[weak self]` |
| `AVAudioRecorder.record` completion           | Да                | —                | `[weak self]` |

### 5. Лучшие практики @escaping в Swift 2026

- **Всегда** ставь `@escaping`, если замыкание вызывается асинхронно или сохраняется  
- **Всегда** `[weak self]` в escaping closure, если захватывается `self`  
- **Всегда** `guard let self else { return }` сразу после входа в closure  
- **Не используй** `[unowned self]` в escaping, если нет 100% гарантии, что self живее замыкания  
- **Для UI** — диспатчи на главный поток: `DispatchQueue.main.async` или `await MainActor.run`  
- **Swift 6 strict concurrency** — захват `self` в escaping closure требует явного `[weak self]` или `@MainActor`  
- **Документируйте** — пиши комментарий «@escaping completion — вызывается после загрузки изображения»

**Короткий девиз 2026**:
> `@escaping` — это когда замыкание говорит: «меня могут вызвать позже, после того как функция уже закончилась».  
> В 2026 году:  
> - `@escaping` + `[weak self]` + `guard let self` — золотой стандарт для асинхронного кода  
> - без `@escaping` — замыкание вызывается синхронно и безопасно захватывает `self` сильно  
> - большинство новых API уже на `async/await` — но старый UIKit и Foundation всё ещё на escaping

Удачи с безопасными и безутечными замыканиями в твоём коде! 🔄