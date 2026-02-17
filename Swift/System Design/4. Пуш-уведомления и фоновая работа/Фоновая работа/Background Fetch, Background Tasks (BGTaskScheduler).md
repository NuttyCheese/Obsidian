#system_design
## Определение

**Background Fetch** и **BGTaskScheduler** — это механизмы iOS, позволяющие **выполнять задачи и обновлять данные приложения в фоне**, даже когда пользователь не открывает приложение.

- **Background Fetch** → старый механизм периодического обновления данных.
    
- **BGTaskScheduler** → современный [[API]] для планирования фоновых задач с гибкими возможностями.
    

> Цель: поддерживать актуальность данных, обновлять кэш и готовить UI без вмешательства пользователя.

---

## 1. Background Fetch

### Принцип работы

- iOS периодически пробуждает приложение в фоне.
    
- Приложение получает ограниченное время для обновления данных.
    
- Система сама решает, **когда пробудить приложение**, исходя из использования батареи и поведения пользователя.
    

### Настройка

1. Включить **Background Modes** → `Background fetch`.
    
2. Реализовать метод в AppDelegate:
    

```swift
func application(_ application: UIApplication,
                 performFetchWithCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {
    fetchData { success in
        completionHandler(success ? .newData : .failed)
    }
}
```

### Особенности

- Время выполнения ограничено (~30 секунд).
    
- Частота вызова зависит от **паттернов использования приложения**.
    
- Подходит для простого обновления данных (новости, статусы заказов).
    

---

## 2. BGTaskScheduler ([[iOS]] 13+)

### Принцип работы

- Позволяет планировать задачи **по расписанию** (`BGAppRefreshTask`) или **для длительной фоновой обработки** (`BGProcessingTask`).
    
- Система решает, **когда именно выполнить задачу**, оптимизируя батарею и ресурсы.
    
- Поддерживает **повторное планирование задач** после выполнения.
    

### Основные типы задач

|Тип|Описание|
|---|---|
|**BGAppRefreshTask**|Короткие задачи для обновления UI и кэша|
|**BGProcessingTask**|Длительные задачи, требующие времени и ресурсов (например, upload файлов)|

---

### Регистрация задач

```swift
import BackgroundTasks

func registerBackgroundTasks() {
    BGTaskScheduler.shared.register(forTaskWithIdentifier: "com.app.refresh", using: nil) { task in
        self.handleAppRefresh(task: task as! BGAppRefreshTask)
    }
    
    BGTaskScheduler.shared.register(forTaskWithIdentifier: "com.app.processing", using: nil) { task in
        self.handleProcessingTask(task: task as! BGProcessingTask)
    }
}
```

### Планирование задачи

```swift
func scheduleAppRefresh() {
    let request = BGAppRefreshTaskRequest(identifier: "com.app.refresh")
    request.earliestBeginDate = Date(timeIntervalSinceNow: 15 * 60) // минимум 15 минут
    try? BGTaskScheduler.shared.submit(request)
}
```

### Обработка задачи

```swift
func handleAppRefresh(task: BGAppRefreshTask) {
    scheduleAppRefresh() // планируем следующую задачу
    
    let queue = OperationQueue()
    queue.maxConcurrentOperationCount = 1
    
    let operation = DataSyncOperation() // ваша операция синхронизации
    
    task.expirationHandler = {
        queue.cancelAllOperations()
    }
    
    operation.completionBlock = {
        task.setTaskCompleted(success: !operation.isCancelled)
    }
    
    queue.addOperation(operation)
}
```

---

## Отличия Background Fetch и BGTaskScheduler

|Характеристика|Background Fetch|BGTaskScheduler|
|---|---|---|
|Минимальная версия iOS|iOS 7+|iOS 13+|
|Тип задач|Короткие обновления|Короткие и длительные задачи|
|Планирование|Система сама определяет частоту|Можно задать `earliestBeginDate` и тип задачи|
|Контроль над задачами|Ограниченный|Более гибкий, можно повторять задачи и комбинировать с очередями операций|

---

## Best Practices

1. **Минимизировать время выполнения** → быстрые операции и локальный кэш.
    
2. **Комбинировать с push-уведомлениями** → silent push для мгновенной синхронизации.
    
3. **Повторное планирование задач** → каждая задача должна планировать следующую.
    
4. **Обрабатывать ошибки и retry** → использовать локальные очереди для синхронизации данных.
    
5. **Использовать правильный тип задачи** → `BGAppRefreshTask` для UI и коротких операций, `BGProcessingTask` для длительных.
    

---

## Применение в мобильных приложениях

- **Новости и контент** → обновление ленты, кэша, изображений.
    
- **Чаты и сообщения** → обновление сообщений и статусов заказов.
    
- **Приложения доставки и e-commerce** → синхронизация заказов и статусов.
    
- **Финансовые приложения** → обновление курсов валют, котировок и транзакций.
    

---

## Итог

- **Background Fetch** → простой способ периодического обновления данных.
    
- **BGTaskScheduler** → современный и гибкий способ планирования фоновых задач.
    
- **Комбинированный подход** → push + BGTaskScheduler обеспечивает актуальность данных и надежность.
    
- Ключевые элементы: **регистрация, планирование, обработка задач, повторное планирование и управление временем выполнения**.
    

---
