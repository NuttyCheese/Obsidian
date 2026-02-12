#memory_control #Swift 
**Autorelease Pool** — механизм отложенного освобождения объектов, унаследованный от [[Objective-C]].

С появлением **[[ARC]]** в 2011 году он стал использоваться редко, но всё ещё важен в специфических случаях.

### Когда нужен @autoreleasepool в [[Swift]] (2026)

| Ситуация                                           | Зачем нужен пул                                | Пример                               |
| -------------------------------------------------- | ---------------------------------------------- | ------------------------------------ |
| Долгий цикл с тысячами временных **ObjC**-объектов | Предотвращает рост памяти до конца [[runloop]] | парсинг строк, изображений, [[JSON]] |
| Работа с legacy ObjC API                           | API возвращает autoreleased объекты            | старые Core Foundation вызовы        |
| Обработка больших данных в фоне                    | Контроль пиков памяти в [[GCD]]/Operation      | загрузка файлов, обработка фото      |
| Бенчмарки / тесты памяти                           | Точное управление освобождением                | Instruments                          |

### Краткие правила

- **Swift-объекты** ([[struct]], [[Array]], [[String]] и т.д.) → **не используют** autorelease pool (COW + ARC)
- **ObjC-объекты** (`NSString`, [[UIImage]], `NSData`, старые Foundation-классы) → **могут** попадать в пул
- `@autoreleasepool { … }` → современный и единственный рекомендуемый способ

### Примеры кода (самые частые кейсы)

#### 1. Тяжёлый цикл с временными строками

```swift
for i in 0..<100_000 {
    @autoreleasepool {
        let _ = NSString(format: "Line %d of log", i)
        // без пула память растёт до конца цикла
    }
}
```

#### 2. Обработка изображений в цикле

```swift
for asset in photoAssets {
    @autoreleasepool {
        guard let image = UIImage(contentsOfFile: asset.path) else { continue }
        // обработка, ресайз, фильтры
        let thumbnail = image.preparingThumbnail(of: CGSize(width: 200, height: 200))
    }
}
```

#### 3. Асинхронная загрузка больших файлов

```swift
DispatchQueue.global(qos: .utility).async {
    @autoreleasepool {
        guard let data = NSData(contentsOfFile: largeFilePath) else { return }
        // парсинг / обработка
    }
}
```

#### 4. Старый Objective-C стиль (legacy)

```objc
NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
// ... работа с объектами
[pool drain]; // или [pool release] в старом коде
```

### Ключевые факты 2026 года

- `@autoreleasepool` — **единственный** рекомендуемый синтаксис
- `NSAutoreleasePool` — устарел, не используй
- В **чистом Swift-коде** без ObjC-интеропа пулы почти не нужны
- Главная польза — **снижение пикового потребления памяти** в циклах
- Без пула в тяжёлом цикле → возможен **memory pressure** и termination приложения

### Короткое правило

> Добавляй `@autoreleasepool { … }` **внутри циклов**, если:
> - создаёшь тысячи **ObjC-объектов** (`NSString`, `NSData`, `UIImage`, …)
> - видишь рост памяти в Instruments без видимой причины

Всё остальное время — ARC делает всё за тебя.
