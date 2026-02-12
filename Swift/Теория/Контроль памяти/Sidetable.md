#memory_control #Swift 
## 📘 Определение
**Side Table** — это **вспомогательная структура**, которую [[Objective-C]] [[Runtime]] создаёт для объекта **только при необходимости** хранить дополнительную информацию, которая не помещается в основной объект (или требует отдельной логики).

### Когда и зачем создаётся Side Table

Side Table появляется **не всегда** и **не сразу**.  
Она создаётся **лениво** в следующих случаях:

| Ситуация                             | Side Table создаётся? | Почему нужен Side Table                                 | Пример кода / сценарий         |
| ------------------------------------ | --------------------- | ------------------------------------------------------- | ------------------------------ |
| Только [[strong]]-ссылки             | ❌                     | —                                                       | `var child: Child?`            |
| Первый [[weak]]-указатель            | ✅                     | Хранит список всех weak-ссылок на объект                | `weak var delegate`            |
| Первый associated object             | ✅                     | Хранит ассоциативные объекты (objc_setAssociatedObject) | `objc_setAssociatedObject`     |
| Переполнение refcount (очень редко)  | ✅                     | Дополнительные биты для refcount > 2³¹–1                | Почти никогда в реальной жизни |
| [[Swift]] [[Runtime]] extra metadata | ✅ (в редких случаях)  | Хранение extra refcount, pinning, etc.                  | Swift 5.9+ internals           |

### Упрощённая схема памяти объекта

```text
Обычный объект (без Side Table)
┌─────────────────────────────────────┐
│ isa pointer (тип + метаданные)      │
│ inline refcount (обычно 32/64 бита) │
│ inline свойства (strong/unowned)    │
└─────────────────────────────────────┘

Объект с Side Table (после первого weak / associated)
┌──────────────────────────────────────┐
│ isa pointer                          │
│ inline refcount (только базовые биты)│
│ inline свойства                      │
│ pointer → Side Table ──────────────┐ |
└──────────────────────────────────────┘
                                      │
                                      ▼
                          ┌──────────────────────────────┐
                          │ Side Table                   │
                          │ - Список weak-ссылок         │
                          │ - Associated objects         │
                          │ - Extra refcount (если нужно)│
                          └──────────────────────────────┘
```

### Под капотом: как работает weak

Когда ты пишешь:

```swift
weak var owner: Owner?
```

Компилятор вставляет:

```swift
objc_storeWeak(&self.owner, owner)
```

А при чтении:

```swift
objc_loadWeak(&self.owner)
```

**objc_storeWeak** делает следующее:
- Если у объекта ещё нет Side Table → **создаёт её**
- Регистрирует адрес поля `&self.owner` в списке weak-ссылок Side Table
- Не увеличивает retain count

**objc_loadWeak**:
- Идёт по адресу объекта → смотрит, есть ли Side Table
- Если есть → проверяет, жив ли объект
- Если жив → возвращает указатель
- Если мёртв → возвращает [[nil]]

### Почему weak дороже strong / [[unowned]]

| Операция   | strong / unowned      | weak                                | Почему weak медленнее      |
| ---------- | --------------------- | ----------------------------------- | -------------------------- |
| Запись     | 1 [[retain]] / ничего | objc_storeWeak (hash + регистрация) | Создание/поиск Side Table  |
| Чтение     | 1 load                | objc_loadWeak (проверка + load)     | Проверка lifetime + atomic |
| При deinit | ничего                | Обход всех weak refs → зануление    | Может быть много weak      |

### Самый частый сценарий создания Side Table в [[iOS]]-приложениях

```swift
class ViewController: UIViewController {
    weak var delegate: SomeDelegate?          // ← первый weak → Side Table создаётся
    var dataSource: DataSource?               // strong — не влияет
}
```

```swift
timer = Timer.scheduledTimer(...) { [weak self] _ in
    self?.updateUI()                          // weak self → Side Table для self
}
```

### Короткий итог (2026)

- Side Table — **не обязательная** структура
- Создаётся **лениво** при:
  - первом `weak`-указателе
  - первом associated object
  - редких случаях refcount overflow
- **weak** — главный потребитель Side Table
- **unowned** и **strong** — **не создают** Side Table
- **[[lazy]] [[var]] + unowned** — идеальный паттерн (нет Side Table, высокая скорость)
- **Замыкания с [weak self]** — самый частый источник Side Table в реальных приложениях

**Главное правило**:
> «Side Table появляется только при первом weak или associated object.  
> Хочешь избежать Side Table → используй unowned там, где lifetime гарантирован (parent → child, lazy после init).  
> Хочешь безопасность → weak (и мирись с небольшим overhead).»
