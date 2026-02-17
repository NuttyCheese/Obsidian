**Tombstone** (надгробие, могильный камень) — это термин, который в программировании (особенно в [[iOS]]/macOS-разработке и [[Objective-C]]/[[Swift]]) имеет несколько очень конкретных значений.

Вот актуальные на 2026 год основные значения слова «tombstone» в контексте Swift/Objective-C:

### 1. **Tombstone в Objective-C runtime (самое частое значение)**

Это специальный объект-«могильный камень», который **заменяет** удалённый (deallocated) объект в памяти, чтобы предотвратить **use-after-free** ошибки и **zombie-объекты**.

Когда в Objective-C включается **zombie mode** (обычно в debug-сборке через Environment Variable `NSZombieEnabled = YES`), runtime вместо освобождения памяти объекта заменяет его указатель на специальный объект-**NSZombie** (tombstone).

- Любое сообщение, отправленное на такой объект → вызывает **EXC_BREAKPOINT** или лог «*** -[NSObject(NSObject) doesNotRecognizeSelector:]»  
- Это **самый мощный инструмент отладки** утечек памяти и use-after-free багов

**Как включить zombies в 2026 году** ([[Xcode]] 18+):

1. Edit Scheme → Run → Diagnostics → Enable Zombie Objects  
2. Или Environment Variables: `NSZombieEnabled = YES`

**Признаки, что ты попал на tombstone**:
```
*** -[__NSArrayM objectAtIndex:]: message sent to deallocated instance 0x1234567890abc
```

### 2. **Tombstone в [[Swift Concurrency]] (реже, но встречается)**

В Swift 6+ с **strict concurrency checking** иногда возникает термин **tombstone** в контексте **Task cancellation** и **actor isolation**.

Когда задача отменяется (`task.cancel()`), её состояние становится «tombstone» — она больше не выполняется, но может оставлять «мёртвые» ссылки или continuation, которые приводят к предупреждениям компилятора.

Это внутренний термин в runtime и компиляторе, в коде ты его не увидишь напрямую.

### 3. **Tombstone в базах данных / [[NoSQL]] / Kafka / Event Sourcing**

В распределённых системах и event-sourcing **tombstone** — это специальное сообщение/запись, которая означает **«удалить этот ключ»**.

- В Kafka: сообщение с **null value** и тем же ключом → tombstone (логический delete)  
- В DynamoDB / Cassandra / Redis Streams — аналогично  
- В event-sourcing: событие `ItemDeleted` или `TombstoneEvent`

Это не связано со Swift напрямую, но часто встречается в бэкендах мобильных приложений.

### 4. **Tombstone в отладке памяти (Instruments / Zombies)**

В Instruments → **Zombies instrument** — это именно то, что «оживляет» tombstone-объекты, чтобы ты мог увидеть, где именно произошёл доступ к уже освобождённому объекту.

### Короткий чек-лист: что делать, если увидел «tombstone» / «zombie»

1. Включи **NSZombieEnabled** в схеме (Run → Diagnostics)  
2. Запусти приложение в debug-режиме  
3. Дождись ошибки «message sent to deallocated instance»  
4. Посмотри стек-трейс — увидишь точное место, где объект был deallocated  
5. Найди [[retain cycle]] / premature dealloc (чаще всего weak/delegate, [weak self], actor isolation)  
6. Исправь → выключи zombies (они замедляют приложение)

**Короткий девиз 2026**:
> «Tombstone / zombie — это когда умерший объект встаёт из могилы и кричит: «ты пытался меня использовать после смерти!»  
> В 2026 году это **лучший друг** при отладке retain cycle и use-after-free.  
> Включи NSZombieEnabled → получи стек-трейс → найди баг → выключи.»
