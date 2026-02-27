**URLSessionTask** — это **абстрактный базовый класс** в [[UIKit]]/[[Foundation]], представляющий **одну конкретную сетевую операцию** (задачу), выполняемую через [[URLSession]].

Все реальные сетевые задачи (dataTask, downloadTask, uploadTask, streamTask) наследуются от `URLSessionTask`.

В 2026 году это всё ещё **основной строительный блок** всей сетевой инфраструктуры iOS/macOS, даже если вы используете [[async]]/[[await]], [[Combine]] или [[Alamofire]] — под капотом почти всегда именно `URLSessionTask`.

### Иерархия классов (актуально 2026)

```
URLSessionTask (абстрактный)
├── URLSessionDataTask          → обычный запрос (data)
├── URLSessionUploadTask        → загрузка файла на сервер
│   └── URLSessionStreamTask    → потоковая загрузка (редко)
├── URLSessionDownloadTask      → скачивание файла
└── URLSessionStreamTask        → работа с потоками (WebSocket-подобные, редко)
```

### Основные свойства и методы URLSessionTask

| Свойство / Метод                            | Тип / Возвращает                                                         | Что даёт / зачем нужно                            | Самый частый сценарий              |
| ------------------------------------------- | ------------------------------------------------------------------------ | ------------------------------------------------- | ---------------------------------- |
| `state`                                     | `URLSessionTask.State` (.running / .suspended / .canceling / .completed) | Текущее состояние задачи                          | Проверка, жива ли задача           |
| `progress`                                  | `Progress`                                                               | Объект Progress с progress, totalUnitCount и т.д. | Отображение прогресс-бара          |
| `countOfBytesReceived` / `countOfBytesSent` | `Int64`                                                                  | Сколько байт получено/отправлено                  | Отладка, логирование               |
| `countOfBytesExpectedToReceive` / `...Send` | `Int64`                                                                  | Ожидаемое количество байт (если известно)         | Показ % загрузки                   |
| `currentRequest` / `originalRequest`        | [[URLRequest]]`?`                                                        | Текущий и исходный запрос                         | Отладка редиректов                 |
| `taskDescription`                           | [[String]]`?`                                                            | Произвольная строка для отладки                   | Логирование                        |
| `cancel()`                                  | `Void`                                                                   | Немедленно отменить задачу                        | Отмена загрузки при уходе с экрана |
| `suspend()` / `resume()`                    | `Void`                                                                   | Приостановить / возобновить задачу                | Экономия трафика в фоне            |
| `resume()`                                  | `Void`                                                                   | **Обязательно** вызывать после создания задачи!   | Без этого задача не стартует       |

### Самые популярные паттерны использования в 2026 году

#### 1. Классический dataTask (самый частый)

```swift
let task = URLSession.shared.dataTask(with: url) { data, response, error in
    // обработка
}
task.resume()  // ← обязательно!
```

#### 2. Современный [[async]]/[[await]] ([[iOS]] 15+)

```swift
@MainActor
func fetchData() async throws -> Data {
    let (data, response) = try await URLSession.shared.data(from: url)
    
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw URLError(.badServerResponse)
    }
    
    return data
}
```

#### 3. Отмена задачи при уходе с экрана (очень важно!)

```swift
class DataLoadingViewController: UIViewController {
    private var dataTask: URLSessionDataTask?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        dataTask = URLSession.shared.dataTask(with: url) { [weak self] data, response, error in
            // обработка
        }
        dataTask?.resume()
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        dataTask?.cancel()
        dataTask = nil
    }
}
```

#### 4. Прогресс загрузки (самый частый сценарий с Progress)

```swift
let task = URLSession.shared.downloadTask(with: url)
task.progress.publisher(for: \.fractionCompleted)
    .receive(on: DispatchQueue.main)
    .sink { progress in
        self.progressView.progress = Float(progress)
    }
    .store(in: &cancellables)
    
task.resume()
```

### Лучшие практики URLSessionTask в 2026 году

- **Всегда вызывайте `resume()`** после создания задачи — без этого ничего не начнётся  
- **Обязательно отменяйте** задачи в [[viewWillDisappear]] / [[deinit]] — иначе утечки трафика и памяти  
- **Используйте `progress`** — он встроенный и очень удобный для UI  
- **Для async/await** — предпочитайте `data(from:)`, `download(from:)`, `upload(for:fromFile:)`  
- **Для Combine** — используйте `dataTaskPublisher` — это самый чистый способ  
- **Для больших файлов** — используйте `downloadTask` вместо `dataTask`  
- **Для upload** — используйте `uploadTask` с прогрессом через делегаты или Progress  
- **Документируйте** — пишите комментарий:

```swift
/// Загрузка данных с возможностью отмены при уходе с экрана
private var dataTask: URLSessionDataTask?
```

**Короткий итог 2026**:
> `URLSessionTask` — это **конкретная сетевая операция** (data, upload, download), выполняемая через `URLSession`.  
> В 2026 году:  
> - ключевые методы — `resume()`, `cancel()`, `suspend()`, `progress`  
> - самый популярный — `dataTask` / `downloadTask` + `async/await`  
> - обязательно отменяйте задачи при уходе с экрана  
> - это **фундаментальный** класс всей сетевой работы в iOS — даже [[Alamofire]]/[[Moya]] под капотом используют именно его  
