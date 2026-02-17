**Autorelease** — механизм из эпохи **Manual Retain-Release (MRR)**, который откладывал вызов `release` до конца текущего **autorelease pool**.

С появлением **[[ARC]]** (Automatic Reference Counting) в 2011 году явное использование `autorelease` и `NSAutoreleasePool` стало почти ненужным, но понимание механизма важно для:

- работы со старым/legacy кодом
- взаимодействия с некоторыми [[API]] [[Foundation]] / [[UIKit]]
- оптимизации в горячих циклах

### 1. Как это работало (до ARC)

```objc
NSString *str = [[[NSString alloc] initWithFormat:@"Hello %@", @"World"] autorelease];
// объект помещается в текущий пул
// в конце пула автоматически вызывается release
```

### 2. Современный эквивалент — `@autoreleasepool`

```swift
@autoreleasepool {
    // создаются тысячи временных объектов
    for _ in 0..<100_000 {
        let _ = NSString(format: "Item %d", arc4random_uniform(1000))
        // каждый NSString помещается в локальный пул
    }
} // здесь все объекты из пула освобождаются
```

Без `@autoreleasepool` в цикле память может расти до конца [[RunLoop]] → пики памяти.

### 3. Когда `@autoreleasepool` всё ещё нужен (2026)

| Сценарий                                                       | Почему нужен пул                               | Пример кода                                     |
| -------------------------------------------------------------- | ---------------------------------------------- | ----------------------------------------------- |
| Долгий цикл с большим количеством временных ObjC-объектов      | Предотвращает пики памяти до конца runloop     | обработка изображений, парсинг [[JSON]]/[[XML]] |
| Работа с legacy [[API]], возвращающими [[autorelease]] объекты | Безопасность при раннем выходе из scope        | старые Core Foundation вызовы                   |
| Обработка больших данных в фоне                                | Контроль пиков памяти в [[GCD]]/OperationQueue | обработка файлов/сетевых ответов                |
| Тесты / бенчмарки                                              | Точный контроль освобождения                   | Instruments                                     |

### 4. Типичные места использования в 2026 году

```swift
// 1. Парсинг большого JSON / XML
@autoreleasepool {
    let data = try Data(contentsOf: url)
    let _ = try JSONSerialization.jsonObject(with: data)
}

// 2. Работа с изображениями в цикле
for asset in assets {
    @autoreleasepool {
        let image = UIImage(contentsOfFile: asset.path)
        // обработка
    }
}

// 3. Долгий фоновый цикл
DispatchQueue.global().async {
    for i in 0..<1_000_000 {
        @autoreleasepool {
            let _ = NSString(format: "Line %d", i)
            // обработка строки
        }
    }
}
```

### 5. Короткое правило 2026 года

- **В обычном коде** — `@autoreleasepool` почти не нужен (ARC + [[Swift]]-объекты не используют пулы)
- **В горячих циклах с ObjC API** — всегда добавляй `@autoreleasepool`
- **Внутри GCD / Operation / [[async]]** — используй, если видишь рост памяти в Instruments
- **Никогда** не используй старый `NSAutoreleasePool` — только `@autoreleasepool { … }`

### 6. Мифы и правда

❌ «ARC полностью заменил autorelease pools»  
→ Нет — пулы всё ещё нужны для временных ObjC-объектов в циклах

❌ «@autoreleasepool замедляет код»  
→ Нет — overhead минимален, а экономия памяти огромна

❌ «В Swift всё [[Value Type]], пулы не нужны»  
→ Swift-объекты ([[Array]], [[String]]) используют COW, но ObjC-интероп всё ещё использует пулы

**Коротко**:  
В 2026 году `@autoreleasepool` — это **инструмент оптимизации памяти** в циклах с большим количеством временных **[[Foundation]]/ObjC** объектов.
