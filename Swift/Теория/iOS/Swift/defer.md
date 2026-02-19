**`defer`** — это ключевое слово в [[Swift]], которое **откладывает выполнение блока кода до момента выхода из текущей области видимости** (scope), независимо от того, как именно этот scope завершается: нормальным выходом, [[return]], [[throw]], [[break]], [[continue]] или даже крашем (fatalError).

Это **самый надёжный** способ гарантированно выполнить очистку ресурсов, закрытие файлов, отписку от уведомлений, остановку таймеров и любые другие завершающие действия.

### 1. Почему defer — один из самых любимых инструментов Swift-разработчиков (2026)

| Ситуация без defer                                   | Проблема                                          | Как defer решает проблему                       |
| ---------------------------------------------------- | ------------------------------------------------- | ----------------------------------------------- |
| Закрытие файла после чтения                          | Забыли вызвать `close()` при `return` или `throw` | `defer { file.close() }` — всегда выполнится    |
| Отмена сетевого запроса при уходе с экрана           | Запрос продолжает работать после `deinit`         | `defer { task.cancel() }` в `viewDidDisappear`  |
| Остановка таймера при уходе с экрана                 | Таймер продолжает тикать → retain cycle           | `defer { timer.invalidate() }`                  |
| Освобождение тяжёлого ресурса ([[AVPlayer]], CoreML) | Память утекает при раннем `return`                | `defer { player?.pause(); player = nil }`       |
| Логирование выхода из функции                        | Нужно писать везде `print("exit")`                | Один `defer { print("exit") }` в начале функции |

### 2. Главные правила defer (запомни навсегда)

1. **defer выполняется в конце scope** — когда функция/метод/блок заканчивается (даже при ошибке)
2. **defer выполняется в обратном порядке** ([[FILO|LIFO]] — последний defer первый)
3. **defer можно использовать несколько раз** в одной области видимости
4. **defer не выполняется**, если программа крашится до выхода из scope (fatalError, signal, segmentation fault)
5. **defer работает только внутри функций/методов/блоков** — нельзя использовать в глобальной области или в свойствах

### 3. Самые популярные и рекомендуемые паттерны 2026 года

#### Паттерн 1: Очистка ресурсов (самый частый)

```swift
func processFile(at url: URL) throws -> Data {
    let fileHandle = try FileHandle(forReadingFrom: url)
    defer { try? fileHandle.close() }  // гарантированно закроем файл
    
    return try fileHandle.readToEnd() ?? Data()
}
```

#### Паттерн 2: Отмена асинхронных задач (очень важно в [[UIKit]]/[[SwiftUI]])

```swift
class ProfileViewController: UIViewController {
    private var profileTask: Task<Void, Never>?
    
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        
        profileTask = Task { [weak self] in
            defer { self?.profileTask = nil }  // очистка ссылки
            
            do {
                let profile = try await fetchProfile()
                await MainActor.run {
                    self?.updateUI(with: profile)
                }
            } catch {
                await MainActor.run {
                    self?.showError(error)
                }
            }
        }
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        profileTask?.cancel()  // отмена при уходе с экрана
    }
}
```

#### Паттерн 3: Несколько defer в одной функции (LIFO порядок)

```swift
func complexOperation() {
    print("1. Начало операции")
    
    defer { print("5. Самый последний defer") }
    defer { print("4. Второй defer") }
    
    print("2. Основная логика")
    
    defer { print("3. Первый defer") }
    
    print("Конец функции")
}

// Вывод:
// 1. Начало операции
// 2. Основная логика
// 3. Первый defer
// 4. Второй defer
// 5. Самый последний defer
// Конец функции
```

#### Паттерн 4: defer + throws (очень частый в async коде)

```swift
func downloadAndProcess() async throws -> Data {
    let (data, _) = try await URLSession.shared.data(from: url)
    
    defer {
        // Освобождаем временные ресурсы
        URLCache.shared.removeAllCachedResponses()
        print("Очистка кэша завершена")
    }
    
    // Долгая обработка
    try await heavyProcessing(data)
    
    return data
}
```

### 4. Лучшие практики defer в Swift 2026

- **Используй defer для всего, что нужно очистить** — файлы, соединения, таймеры, observers, tasks  
- **Пиши defer как можно раньше** в функции — сразу после открытия ресурса  
- **Несколько defer** — последний написанный выполнится первым (LIFO)  
- **Не используй defer для логики** — только для cleanup (это антипаттерн)  
- **В async функциях** — defer работает так же, но часто комбинируется с `Task`/`await MainActor.run`  
- **Swift 6 strict concurrency** — defer полностью безопасен, но код внутри defer должен учитывать актор  
- **Документируйте** — пиши комментарий «defer — гарантированное закрытие файла при любом выходе»

**Короткий девиз 2026**:
> `defer` — это «гарантированное завершение» в конце scope.  
> В 2026 году используй его **везде**, где открываешь ресурс: файлы, сессии, таймеры, задачи, observers.  
> Пиши defer **сразу после открытия** ресурса — это самый надёжный и читаемый стиль.

Удачи с безопасной и гарантированной очисткой ресурсов в твоём коде! 🧹