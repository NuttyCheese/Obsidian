#didreceivememorywarning #uiviewcontroller #memory-management #ios #swift #low-memory #app-lifecycle #performance #leak-prevention #background #multitasking

---
**(метод предупреждения о нехватке памяти)**

**didReceiveMemoryWarning()** — это метод жизненного цикла [[UIViewController]], который система вызывает, когда **устройство испытывает нехватку оперативной памяти** и просит ваше приложение **освободить как можно больше ресурсов как можно быстрее**.

Этот метод существует с самых первых версий [[iOS]] и остаётся **критически важным** в 2026 году, несмотря на то, что современные устройства имеют 8–16 ГБ RAM.  
Причины: 

- фоновые приложения
- многозадачность (Split View, Stage Manager)
- утечки памяти в вашем коде
- тяжёлые ресурсы (большие изображения, видео, 3D-модели, кэш)
- iOS может убить приложение, если оно не отреагирует

### Когда вызывается didReceiveMemoryWarning()

Система вызывает его в следующих случаях (примерный порядок по вероятности):

1. Устройство в **низком состоянии памяти** (low memory pressure)
2. Приложение в **фоне** и система хочет освободить RAM
3. Пользователь открывает много приложений / вкладок в Safari
4. Приложение использует **много памяти** (например, 300+ МБ на iPhone)
5. iOS готовится показать **предупреждение "Memory Almost Full"** или убить приложение

**Важно:**  
- Метод вызывается **только на UIViewController** (не на [[UIView]], не на модели)
- Вызывается **асинхронно** и **может быть вызван многократно**
- **Не гарантируется**, что он будет вызван перед тем, как приложение убьют (jetsam)

### Что делать внутри didReceiveMemoryWarning() (рекомендации 2026)

**Цель:** освободить как можно больше памяти **без нарушения пользовательского опыта**.

**Хорошие практики:**

```swift
override func didReceiveMemoryWarning() {
    super.didReceiveMemoryWarning()
    
    // 1. Очистка кэша изображений (самое частое)
    ImageCache.shared.removeAllObjects()
    URLCache.shared.removeAllCachedResponses()
    
    // 2. Выгрузка тяжёлых view / layers
    if isViewLoaded {
        heavyImageView.image = nil
        complexLayer.contents = nil
        webView?.stopLoading()
        webView?.loadHTMLString("", baseURL: nil)
    }
    
    // 3. Отмена ненужных операций
    dataTask?.cancel()
    dataTask = nil
    
    operationQueue?.cancelAllOperations()
    
    // 4. Очистка временных данных
    temporaryLargeArray = nil
    largeDictionary.removeAll()
    
    // 5. Логирование для отладки (только в debug)
    #if DEBUG
    print("Memory warning received in \(Self.self)")
    #endif
}
```

**Что НЕ делать в didReceiveMemoryWarning():**

- Не выполнять тяжёлые вычисления
- Не писать на диск (это только усугубит ситуацию)
- Не показывать UI (алерты, модалки)
- Не пытаться "спасти" приложение — система уже в стрессе

### Типичные места, где нужно чистить память

| Тип ресурса                                                   | Как чистить в didReceiveMemoryWarning()                        | Почему это важно                   |
| ------------------------------------------------------------- | -------------------------------------------------------------- | ---------------------------------- |
| Кэш изображений ([[SDWebImage]], [[Kingfisher]], [[NSCache]]) | `cache.removeAllObjects()`                                     | Самый большой потребитель RAM      |
| [[URLCache]]                                                  | `URLCache.shared.removeAllCachedResponses()`                   | Кэшированные ответы [[API]]        |
| [[WKWebView]] / [[UIWebView]]                                 | `stopLoading()`, `loadHTMLString("", baseURL: nil)`            | Веб-контент может держать сотни МБ |
| [[AVPlayer]] / [[AVAudioPlayer]]                              | `pause()`, `replaceCurrentItem(with: nil)`                     | Буферы аудио/видео                 |
| [[Core Data]] / [[Realm]] большие выборки                     | Сбрасывать fetchedResultsController, очищать временные объекты | Много [[NSManagedObject]] в памяти |
| [[Combine]] / [[RxSwift]] подписки                            | Хранить в `Set<AnyCancellable>`, очищать в deinit              | Редко, но бывает                   |
| Большие массивы / словари                                     | `removeAll()` или присвоить `nil`                              | Временные данные                   |

### Современные паттерны 2026 года

1. **Автоматическая очистка через NotificationCenter**

```swift
NotificationCenter.default.addObserver(
    self,
    selector: #selector(handleMemoryWarning),
    name: UIApplication.didReceiveMemoryWarningNotification,
    object: nil
)

@objc private func handleMemoryWarning() {
    // Вызывается даже если контроллер не в top-most
    clearHeavyResources()
}
```

2. **Использование `os_proc_available_memory()`** (iOS 13+)

```swift
import os

if os_proc_available_memory() < 100 * 1024 * 1024 {  // < 100 МБ
    clearHeavyResources()
}
```

3. **Combine + memory warning**

```swift
NotificationCenter.default.publisher(for: UIApplication.didReceiveMemoryWarningNotification)
    .sink { [weak self] _ in
        self?.clearCache()
    }
    .store(in: &cancellables)
```

### Лучшие практики didReceiveMemoryWarning() в 2026 году

- **Вызывайте super** — обязательно!
- **Очищайте только то, что можно легко восстановить** (кэш, изображения, временные данные)
- **Не очищайте критические данные** (модель пользователя, текущий черновик)
- **Логируйте** в debug — чтобы понимать, когда и почему вызывается
- **Тестируйте** через Instruments → Allocations → Simulate Memory Warning
- **Для SwiftUI** — аналог `onReceive(NotificationCenter.default.publisher(for: UIApplication.didReceiveMemoryWarningNotification))`
- **Документируйте** — пишите комментарий:

```swift
override func didReceiveMemoryWarning() {
    super.didReceiveMemoryWarning()
    
    // Освобождаем некритичные ресурсы при нехватке памяти
    clearCachesAndHeavyResources()
}
```

**Короткий итог 2026**:
> **didReceiveMemoryWarning()** — метод, который система вызывает, когда **устройству не хватает RAM**, и просит ваше приложение **освободить память**.  
> Junior: "Место, где чистишь кэш, когда iPhone говорит 'мало памяти'".  
> Middle: вызывается на каждом UIViewController → очищай кэш изображений, отменяй задачи, nil'ай тяжёлые объекты.  
> Senior: комбинируй с `UIApplication.didReceiveMemoryWarningNotification`, проверяй `os_proc_available_memory()`, тестируй через Instruments, не делай ничего тяжёлого внутри.  
