#objective_c #ios #memory_management #release #retain_release #arc #object_lifecycle #heap #manual_memory_management
В **[[Swift]]** ключевое слово **`release`** **не используется** и **не существует** в языке.  
Управление памятью полностью автоматическое благодаря **[[ARC]]** (Automatic Reference Counting).

### Swift vs Objective-C: сравнение

| Аспект               | Swift (с ARC)                   | [[Objective-C]] (MRR — Manual Retain-Release)       | Objective-C (с ARC)            |
| -------------------- | ------------------------------- | --------------------------------------------------- | ------------------------------ |
| Вызов `release`      | **Никогда** не нужен            | Обязателен вручную                                  | Не нужен (как в Swift)         |
| Вызов [[retain]]     | Не нужен                        | Обязателен вручную                                  | Не нужен                       |
| `autorelease`        | Не используется                 | Часто использовался                                 | Не нужен, но пулы всё ещё есть |
| Управление памятью   | Полностью автоматическое (ARC)  | Ручное                                              | Автоматическое (ARC)           |
| Счётчик ссылок       | Управляется компилятором        | Управляется разработчиком                           | Управляется компилятором       |
| `dealloc` / `deinit` | `deinit` — автоматический вызов | [[dealloc]] — вызывается вручную после `release`    | `dealloc` — автоматический     |
| Циклы ссылок         | Разрываются `weak` / `unowned`  | Разрываются `assign` / `weak` / `unsafe_unretained` | То же, что в Swift             |

### Почему в Swift нет `release`?

ARC — это **compile-time механизм**.  
Компилятор сам вставляет вызовы `objc_retain` / `objc_release` / `objc_storeStrong` в нужных местах.  
Разработчик **не может** (и **не должен**) вызывать `release` вручную — это приведёт к **двойному освобождению** и крашу.

### Когда вы всё ещё можете увидеть `release` в 2026 году

1. **Legacy Objective-C код** (без ARC или с флагом `-fno-objc-arc`)

```objc
- (void)dealloc {
    [self.myObject release];
    [super dealloc];
}
```

2. **Взаимодействие с низкоуровневым Core Foundation** (CFRetain / CFRelease)

```swift
let cfString = CFStringCreateWithCString(nil, "Hello", kCFStringEncodingUTF8)!
CFRelease(cfString) // вручную, потому что это Core Foundation объект
```

3. **Ручное управление памятью в C/C++ коде** (редко)

### Ключевые правила для современного Swift (2026)

- **Никогда** не вызывайте `release`, `retain`, [[autorelease]] в Swift-коде
- Если видите такие вызовы — это **legacy** или **низкоуровневый** код
- Используйте `weak` / `unowned` для разрыва циклов
- Проверяйте [[deinit]] с логами — если не вызывается → цикл сильных ссылок
- Для Core Foundation объектов (CF...) — используйте `CFRelease` вручную

### Короткий итог

- В **чистом Swift** — `release` **не существует** и **не нужен**
- ARC делает всё автоматически и **детерминированно**
- `release` встречается только в:
  - старом Objective-C коде
  - Core Foundation (CFRelease)
  - смешанных проектах с `-fno-objc-arc`
- **Главное правило**:  
  «Если ты пишешь `release` в Swift-коде в 2026 году — ты, скорее всего, делаешь что-то не так.»
