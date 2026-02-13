#swift #memory_management #objective_c #dealloc #ios_development
# Освобождение памяти в Swift: deinit

В Swift вместо метода `dealloc` из [[Objective-C]] используется **деинициализатор** — блок `deinit`.

### Основные отличия от Objective-C

| Характеристика      | Objective-C (`dealloc`)                     | [[Swift]] ([[deinit]])                         |
| ------------------- | ------------------------------------------- | ---------------------------------------------- |
| Название            | `- (void)dealloc`                           | `deinit { … }`                                 |
| Вызов вручную       | Обязательно `[super dealloc];` в конце      | **Никогда** не вызывается вручную              |
| Вызов super         | Обязателен                                  | Автоматически вызывается сам                   |
| Когда вызывается    | После последнего [[release]]                | Когда [[ARC]] обнуляет сильные ссылки (RC = 0) |
| Детерминированность | Да, но порядок может быть сложным           | **Строго детерминирован**                      |
| Доступ к self       | Нельзя использовать после `[super dealloc]` | Можно использовать до конца блока `deinit`     |

### Пример deinit

```swift
class FileHandler {
    let path: String
    private var file: UnsafeMutablePointer<FILE>?
    
    init(path: String) {
        self.path = path
        file = fopen(path, "w")
        print("Файл открыт: \(path)")
    }
    
    deinit {
        if let file = file {
            fclose(file)
            print("Файл закрыт: \(path)")
        }
    }
}

var handler: FileHandler? = FileHandler(path: "log.txt")
// ... использование
handler = nil  // → deinit вызван сразу после обнуления ссылки
```

Вывод:
```
Файл открыт: log.txt
Файл закрыт: log.txt
```

### Важные правила и лучшие практики (2026)

1. **deinit вызывается только для классов**  
   [[struct]], [[enum]], [[tuple]] — [[value type]] → не имеют `deinit`.

2. **Порядок вызовов deinit**  
   Сначала `deinit` дочернего класса → потом родительского (обратный порядок инициализации).

```swift
class Parent {
    deinit { print("Parent deinit") }
}

class Child: Parent {
    deinit { print("Child deinit") }
}

var child: Child? = Child()
child = nil
// Вывод:
// Child deinit
// Parent deinit
```

3. **Что можно и нельзя делать в deinit**

Можно:
- Закрывать файлы, сокеты, соединения с БД
- Отписываться от [[NotificationCenter]]
- Освобождать ресурсы (Core Graphics, Metal и т.д.)
- Логировать

Нельзя:
- Доступаться к [[weak]] ссылкам (могут быть уже [[nil]])
- Вызывать методы, которые могут удерживать [[self]]
- Запускать асинхронные операции (они не успеют завершиться)

4. **Частые места использования deinit в iOS**

```swift
class LocationManager: NSObject, CLLocationManagerDelegate {
    private let manager = CLLocationManager()
    
    override init() {
        super.init()
        manager.delegate = self
    }
    
    deinit {
        manager.stopUpdatingLocation()
        print("LocationManager уничтожен")
    }
}
```

```swift
class Observer {
    init() {
        NotificationCenter.default.addObserver(self, selector: #selector(handle), name: .UIApplicationDidEnterBackground, object: nil)
    }
    
    @objc func handle() { /* ... */ }
    
    deinit {
        NotificationCenter.default.removeObserver(self)
    }
}
```

### Короткий итог

- `deinit` — это **Swift-аналог** `dealloc`
- Вызывается **автоматически** при RC = 0
- **Не нужно** вызывать `super.deinit`
- **Стабильный порядок** вызовов (сначала дочерние классы)
- Используй для: закрытия ресурсов, отписок, логирования
- **Никогда** не запускай в `deinit` асинхронные операции

**Главное правило**:
> «Если объект держит ресурсы (файлы, сокеты, наблюдатели, таймеры) — всегда пиши `deinit` для их безопасного освобождения.»
