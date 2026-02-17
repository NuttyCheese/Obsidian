#memory_management #objective_c #reference_counting #retain #ios_development
В **[[Swift]]** ключевое слово **`retain`** **не существует** и **никогда не используется**.  
Управление памятью полностью автоматическое благодаря **[[ARC]]** (Automatic Reference Counting).

### Swift vs Objective-C: сравнение

| Действие / концепция          | Swift (с ARC)                                 | [[Objective-C]] (до ARC / MRR)         | Objective-C (с ARC)      |
| ----------------------------- | --------------------------------------------- | -------------------------------------- | ------------------------ |
| Увеличение счётчика ссылок    | Автоматически при создании сильной ссылки     | `[obj retain]`                         | Автоматически            |
| Уменьшение счётчика ссылок    | Автоматически при [[nil]] или выходе из scope | `[obj release]`                        | Автоматически            |
| Отложенное освобождение       | Не нужно                                      | `[obj autorelease]`                    | `@autoreleasepool { … }` |
| Вызов `retain` / [[release]]  | **Запрещено** и **не компилируется**          | Обязательно вручную                    | Не нужно                 |
| Счётчик ссылок (retain count) | Управляется компилятором                      | Управляется разработчиком              | Управляется компилятором |
| Правила памяти                | [[weak]] / [[unowned]] для разрыва циклов     | `retain` / `release` / [[autorelease]] | То же, что в Swift       |

### Почему в Swift нет `retain` / `release`

ARC — это **compile-time** механизм:  
компилятор сам вставляет вызовы `objc_retain` / `objc_release` / `objc_storeStrong` в нужных местах.  
Разработчик **не имеет доступа** к этим функциям в обычном Swift-коде — это **низкоуровневый API** runtime.

Попытка вызвать `retain` / `release` вручную в Swift приведёт к **ошибке компиляции** или **двойному освобождению** (crash).

### Когда вы всё ещё можете увидеть `retain` / `release` в 2026 году

1. **Legacy Objective-C код** без ARC (файлы с флагом `-fno-objc-arc`)

```objc
- (void)dealloc {
    [self.myString release];
    [super dealloc];
}
```

2. **Низкоуровневый Core Foundation** (CFRetain / CFRelease)

```swift
let cfData = CFDataCreate(nil, bytes, length)!
CFRelease(cfData)  // вручную, потому что это CF-объект
```

3. **Смешанные проекты** (Objective-C + Swift) с legacy-файлами

### Ключевые правила для современного Swift (2026)

- **Никогда** не пишите `retain`, `release`, `autorelease` в Swift-коде
- Если видите такие вызовы — это либо **старый Objective-C**, либо **Core Foundation**
- Для управления памятью используйте:
  - `weak var` / `unowned` — для разрыва циклов
  - `[weak self]` в замыканиях — всегда по умолчанию
  - `deinit` с логами — для проверки освобождения
- Core Foundation объекты (CFString, CFArray и т.д.) — **вручную** `CFRelease`

### Короткий итог

- В **чистом Swift** → `retain` / `release` **не существуют**
- ARC полностью заменяет их — компилятор делает всё сам
- `retain` / `release` встречаются только в:
  - старом Objective-C коде
  - Core Foundation (CFRelease / CFRetain)
  - смешанных проектах с `-fno-objc-arc`
- **Главное правило 2026**:
  > «Если ты пишешь `retain` или `release` в Swift-коде — ты либо работаешь с legacy, либо с Core Foundation, либо делаешь ошибку.»
