Вот **полное, подробное и максимально насыщенное** руководство по **`DispatchWorkItem`** в Swift и Grand Central Dispatch (GCD) — актуально на февраль 2026 года.

### 1. Что такое DispatchWorkItem и зачем он нужен

**DispatchWorkItem** — это **объект-задача**, который:

- инкапсулирует блок кода (closure)  
- может быть **отменён** до или во время выполнения  
- может иметь **зависимости** и **уведомления** о завершении  
- позволяет **контролировать** выполнение задачи в GCD более гибко, чем просто `async { ... }`

Главные преимущества над обычным замыканием в 2026 году:

- **Отмена** задачи (`cancel()`) — самая востребованная фича  
- Возможность **проверять статус** (`isCancelled`, `isExecuting`)  
- Уведомление о завершении/отмене (`notify`)  
- Поддержка **кастомного QoS** и **флагов**  
- Интеграция с **DispatchGroup**, **OperationQueue** и **Swift Concurrency**

**Самые частые сценарии использования в iOS-приложениях 2026**:

- Отмена долгих операций при уходе с экрана  
- Дебонсинг / throttle ввода пользователя  
- Защита от множественных нажатий кнопки  
- Ограничение частоты запросов к API  
- Тестирование асинхронного кода с таймаутами  
- Переходные сценарии при миграции GCD → Swift Concurrency

### 2. Основные свойства и методы DispatchWorkItem

| Свойство / Метод                     | Что делает                                                                 | Тип возвращаемого значения | Самый частый use-case |
|--------------------------------------|-----------------------------------------------------------------------------|-----------------------------|-----------------------|
| `init(qos:flags:block:)`             | Создаёт WorkItem с приоритетом и флагами                                   | DispatchWorkItem            | Всегда с `.userInitiated` / `.utility` |
| `cancel()`                           | Помечает задачу как отменённую                                             | Void                        | Отмена при уходе с экрана |
| `isCancelled`                        | true, если задача отменена                                                  | Bool                        | Проверка внутри замыкания |
| `isExecuting`                        | true, если задача уже начала выполняться                                    | Bool                        | Редко                     |
| `notify(queue:execute:)`             | Выполняет замыкание после завершения/отмены задачи                          | Void                        | Обновление UI после задачи |
| `perform()`                          | Немедленно выполняет задачу в текущем потоке                                | Void                        | Редко                     |

### 3. Самые популярные и правильные шаблоны использования в 2026

#### Шаблон 1 — Отмена задачи при уходе с экрана (самый частый)

```swift
class ProfileViewController: UIViewController {
    private var profileTask: DispatchWorkItem?
    
    func loadProfile() {
        profileTask?.cancel()
        
        let workItem = DispatchWorkItem { [weak self] in
            guard let self else { return }
            
            if workItem.isCancelled { return }
            
            let profile = fetchProfileSync() // тяжёлая работа
            
            DispatchQueue.main.async {
                if workItem.isCancelled { return }
                self.nameLabel.text = profile.name
                self.avatarImageView.image = profile.avatar
            }
        }
        
        profileTask = workItem
        
        DispatchQueue.global(qos: .userInitiated).async(execute: workItem)
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        profileTask?.cancel()
    }
}
```

#### Шаблон 2 — Дебонсинг поиска (очень актуально)

```swift
class SearchViewController: UIViewController {
    private var searchWorkItem: DispatchWorkItem?
    
    @IBOutlet weak var searchBar: UISearchBar!
    
    func searchBar(_ searchBar: UISearchBar, textDidChange searchText: String) {
        searchWorkItem?.cancel()
        
        let workItem = DispatchWorkItem { [weak self] in
            guard let self else { return }
            
            let results = searchSync(query: searchText)
            
            DispatchQueue.main.async {
                self.tableView.reloadData(with: results)
            }
        }
        
        searchWorkItem = workItem
        
        DispatchQueue.global(qos: .userInitiated).asyncAfter(
            deadline: .now() + 0.4,
            execute: workItem
        )
    }
}
```

#### Шаблон 3 — Защита от спама нажатий кнопки

```swift
@IBAction func submitButtonTapped() {
    submitButton.isEnabled = false
    
    let workItem = DispatchWorkItem { [weak self] in
        guard let self else { return }
        
        let success = submitFormSync()
        
        DispatchQueue.main.async {
            self.submitButton.isEnabled = true
            
            if success {
                self.showSuccessAlert()
            } else {
                self.showErrorAlert()
            }
        }
    }
    
    DispatchQueue.global(qos: .userInitiated).async(execute: workItem)
    
    // Через 1.5 секунды кнопка снова активна, даже если задача не закончилась
    DispatchQueue.main.asyncAfter(deadline: .now() + 1.5) {
        if !workItem.isFinished {
            self.submitButton.isEnabled = true
        }
    }
}
```

### 4. DispatchWorkItem vs современные альтернативы (2026 сравнение)

| Механизм                  | Отмена задачи | Точность задержки | Интеграция с async/await | Рекомендация 2026 | Когда всё ещё использовать |
|---------------------------|---------------|--------------------|---------------------------|-------------------|-----------------------------|
| DispatchWorkItem          | Да            | ±10–50 мс          | Плохо                     | Legacy            | Поддержка старого кода      |
| Task + Task.sleep         | Да (cancel)   | ±5–20 мс           | Отлично                   | Основной выбор    | Новый код                   |
| Timer.scheduledTimer      | Да            | ±50 мс             | Плохо                     | Редко             | Повторяющиеся таймеры       |
| async let + Task.sleep    | Да            | ±5–20 мс           | Отлично                   | Параллельные задержки | Современные сценарии        |

**Вывод 2026**:  
`DispatchWorkItem` — это **уже legacy-инструмент**.  
В новом коде почти всегда лучше использовать **`Task { try await Task.sleep(...) }`** + **`cancel()`**.

### 5. Типичные ошибки и ловушки 2026 года

| Ошибка                                      | Последствия                              | Как избежать |
|---------------------------------------------|------------------------------------------|--------------|
| Забыть отменить при уходе с экрана          | Накопление задач → утечки / задержки     | Всегда `cancel()` в `viewWillDisappear` / `deinit` |
| Множество `asyncAfter` без отмены           | Накопление задач → задержки / утечки     | Использовать `Task` с `cancel()` |
| `sync` внутри `DispatchWorkItem` на main    | Deadlock                                 | Никогда не использовать `sync` на main |
| Задержка > 5–10 секунд без индикатора       | Пользователь думает, что приложение зависло | Показывать прогресс / отмену |
| Использование в тестах без таймаута         | Тест зависает                            | Использовать `XCTestExpectation` |

### 6. Лучшие практики 2026 года

- **Переходите на Swift Concurrency** — `Task.sleep` почти всегда лучше `asyncAfter`  
- **Отмена** — всегда добавляйте возможность отмены задачи (`cancel()`)  
- **Точность** — для анимаций используйте кадры, а не секунды  
- **Таймауты** — избегайте задержек > 1–2 секунд без индикатора прогресса  
- **Тесты** — используйте `XCTestExpectation` + `waitForExpectations` или `async` тесты  
- **Swift 6 strict concurrency** — часто подсвечивает небезопасные задержки вне `@MainActor`  
- **Мониторинг** — используйте `os_signpost` для отслеживания реального времени задержки в Instruments

**Короткий девиз 2026**:
> «DispatchWorkItem — это когда нужно отложить задачу и иметь возможность её отменить.  
> В 2026 году это уже legacy-инструмент.  
> Новый стандарт — Task { try await Task.sleep(...) } + cancel().  
> Если ты всё ещё пишешь DispatchWorkItem в новом коде — спроси себя: «А точно ли это нужно?»»

Удачи с отзывчивым, отменяемым и современным кодом задержек в Swift! ⏳