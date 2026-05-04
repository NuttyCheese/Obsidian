#autolayout #nsanchor #uikit #layout #constraints #ios #swift

---

## [[Auto Layout]] vs [[NSLayoutAnchor]] — Эволюция создания констрейнтов

### Определение

**Auto Layout** — это система динамической верстки в [[UIKit]], которая описывает отношения между элементами интерфейса с помощью математических выражений (констрейнтов). **NSLayoutAnchor** — это [[API]] (появился в [[iOS]] 9), который предоставляет типобезопасный и читаемый способ создания этих констрейнтов, являясь надстройкой над классическим API.

Ключевое понимание: **NSLayoutAnchor не заменяет Auto Layout, а является современным интерфейсом для работы с ним**.

---

### Сравнение API

| Характеристика       | Классический Auto Layout ([[NSLayoutConstraint]])                                 | NSLayoutAnchor                                                       |
| -------------------- | --------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| **Появление**        | iOS 6.0 (2012)                                                                    | iOS 9.0 (2015)                                                       |
| **Синтаксис**        | Громоздкий, с формулой `item.attribute = multiplier * item2.attribute + constant` | Читаемый, цепочечный (`view.leadingAnchor.constraint(equalTo: ...)`) |
| **Типобезопасность** | Низкая (опечатки в атрибутах ловятся в runtime)                                   | Высокая (ошибки на этапе компиляции)                                 |
| **Активация**        | `isActive = true` или `addConstraints()`                                          | Аналогично                                                           |
| **Совместимость**    | Старые проекты, сложные мультипликативные связи                                   | Все современные проекты                                              |
| **Читаемость**       | Низкая                                                                            | Высокая                                                              |

---

### 1. Классический Auto Layout (NSLayoutConstraint)

#### Базовый синтаксис

```swift
import UIKit

class ClassicViewController: UIViewController {
    let redView = UIView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        redView.backgroundColor = .systemRed
        redView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(redView)
        
        // Классический констрейнт (громоздкий)
        let leadingConstraint = NSLayoutConstraint(
            item: redView,
            attribute: .leading,
            relatedBy: .equal,
            toItem: view,
            attribute: .leadingMargin,
            multiplier: 1.0,
            constant: 20.0
        )
        
        let topConstraint = NSLayoutConstraint(
            item: redView,
            attribute: .top,
            relatedBy: .equal,
            toItem: view,
            attribute: .topMargin,
            multiplier: 1.0,
            constant: 20.0
        )
        
        let widthConstraint = NSLayoutConstraint(
            item: redView,
            attribute: .width,
            relatedBy: .equal,
            toItem: nil,
            attribute: .notAnAttribute,
            multiplier: 1.0,
            constant: 100.0
        )
        
        let heightConstraint = NSLayoutConstraint(
            item: redView,
            attribute: .height,
            relatedBy: .equal,
            toItem: nil,
            attribute: .notAnAttribute,
            multiplier: 1.0,
            constant: 100.0
        )
        
        NSLayoutConstraint.activate([
            leadingConstraint, topConstraint, widthConstraint, heightConstraint
        ])
    }
}
```

#### Проблемы классического API

1. **Многословность:** Даже для простого констрейнта нужно много кода.
2. **Низкая типобезопасность:** Можно перепутать `attribute` и `toItem.attribute`.
3. **Сложность чтения:** Трудно понять, что именно делает констрейнт.
4. **Ошибки в runtime:** Опечатка в `attribute` вызовет crash только при выполнении.

---

### 2. NSLayoutAnchor (Современный API)

#### Базовый синтаксис

```swift
class AnchorViewController: UIViewController {
    let redView = UIView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        redView.backgroundColor = .systemRed
        redView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(redView)
        
        // NSLayoutAnchor — читаемо и типобезопасно
        NSLayoutConstraint.activate([
            redView.leadingAnchor.constraint(equalTo: view.leadingMarginAnchor, constant: 20),
            redView.topAnchor.constraint(equalTo: view.topMarginAnchor, constant: 20),
            redView.widthAnchor.constraint(equalToConstant: 100),
            redView.heightAnchor.constraint(equalToConstant: 100)
        ])
    }
}
```

#### Доступные Anchor'ы

| Anchor                    | Тип                     | Описание                       |
| ------------------------- | ----------------------- | ------------------------------ |
| **`leadingAnchor`**       | [[NSLayoutXAxisAnchor]] | Левый край (с учётом RTL)      |
| **`trailingAnchor`**      | `NSLayoutXAxisAnchor`   | Правый край (с учётом RTL)     |
| **`leftAnchor`**          | `NSLayoutXAxisAnchor`   | Левый край (без учёта RTL)     |
| **`rightAnchor`**         | `NSLayoutXAxisAnchor`   | Правый край (без учёта RTL)    |
| **`topAnchor`**           | [[NSLayoutYAxisAnchor]] | Верхний край                   |
| **`bottomAnchor`**        | `NSLayoutYAxisAnchor`   | Нижний край                    |
| **`centerXAnchor`**       | `NSLayoutXAxisAnchor`   | Центр по X                     |
| **`centerYAnchor`**       | `NSLayoutYAxisAnchor`   | Центр по Y                     |
| **`widthAnchor`**         | [[NSLayoutDimension]]   | Ширина                         |
| **`heightAnchor`**        | `NSLayoutDimension`     | Высота                         |
| **`firstBaselineAnchor`** | `NSLayoutYAxisAnchor`   | Первая базовая линия текста    |
| **`lastBaselineAnchor`**  | `NSLayoutYAxisAnchor`   | Последняя базовая линия текста |

---

### Преимущества NSLayoutAnchor

#### 1. **Типобезопасность**

```swift
// ✅ Компилятор проверит, что anchor'ы совместимы
redView.leadingAnchor.constraint(equalTo: view.leadingAnchor)

// ❌ Ошибка компиляции: нельзя смешивать XAxis и YAxis
// redView.leadingAnchor.constraint(equalTo: view.topAnchor)
```

#### 2. **Читаемость**

```swift
// Классический API — трудно читать
NSLayoutConstraint(item: label, attribute: .firstBaseline,
                   relatedBy: .equal,
                   toItem: textField, attribute: .firstBaseline,
                   multiplier: 1.0, constant: 0)

// NSLayoutAnchor — читается как предложение
label.firstBaselineAnchor.constraint(equalTo: textField.firstBaselineAnchor)
```

#### 3. **Меньше кода**

```swift
// Классический — 10 строк
let constraint = NSLayoutConstraint(...)
constraint.isActive = true

// NSLayoutAnchor — 1 строка
label.topAnchor.constraint(equalTo: button.bottomAnchor, constant: 8).isActive = true
```

#### 4. **Автодополнение в Xcode**

При вводе `view.leadingAnchor.` Xcode подскажет доступные методы (`.constraint(equalTo:)`, `.constraint(lessThanOrEqualTo:)` и т.д.).

---

### Сложные констрейнты с NSLayoutAnchor

#### 1. **Мультипликативные отношения**

```swift
// Ширина = половина ширины родителя
redView.widthAnchor.constraint(equalTo: view.widthAnchor, multiplier: 0.5)

// Высота = 2 × ширина
redView.heightAnchor.constraint(equalTo: redView.widthAnchor, multiplier: 2.0)
```

#### 2. **Неравенства**

```swift
label.leadingAnchor.constraint(greaterThanOrEqualTo: view.leadingAnchor, constant: 20)
label.trailingAnchor.constraint(lessThanOrEqualTo: view.trailingAnchor, constant: -20)
```

#### 3. **Приоритеты**

```swift
let constraint = label.widthAnchor.constraint(equalToConstant: 200)
constraint.priority = .defaultHigh
constraint.isActive = true
```

#### 4. **Комбинация с UILayoutGuide**

```swift
let guide = view.safeAreaLayoutGuide
button.bottomAnchor.constraint(equalToSystemSpacingBelow: guide.bottomAnchor, multiplier: 1.0)
```

---

### VFL (Visual Format Language) vs NSLayoutAnchor

| Характеристика | VFL | NSLayoutAnchor |
|---|---|---|
| **Краткость** | Высокая | Средняя |
| **Типобезопасность** | Низкая (строки) | Высокая |
| **Читаемость сложных связей** | Хорошая | Отличная |
| **Поддержка Xcode** | Ограниченная | Отличная |

```swift
// VFL
let views = ["button": button]
let constraints = NSLayoutConstraint.constraints(
    withVisualFormat: "H:|-20-[button(100)]-20-|",
    metrics: nil,
    views: views
)

// NSLayoutAnchor
button.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20).isActive = true
button.widthAnchor.constraint(equalToConstant: 100).isActive = true
button.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20).isActive = true
```

---

### Когда использовать каждый подход

| Сценарий | Рекомендация |
|---|---|
| **Новый проект** | Только NSLayoutAnchor |
| **Поддержка iOS 8-** | Классический API (но iOS 8 уже не поддерживается) |
| **Очень сложные мультипликативные зависимости** | Классический API (редко) |
| **Быстрый прототип** | NSLayoutAnchor + удобные расширения |
| **Динамические констрейнты в runtime** | NSLayoutAnchor (легче изменять) |

---

### Расширения для упрощения NSLayoutAnchor

```swift
extension UIView {
    func anchor(
        top: NSLayoutYAxisAnchor? = nil,
        leading: NSLayoutXAxisAnchor? = nil,
        bottom: NSLayoutYAxisAnchor? = nil,
        trailing: NSLayoutXAxisAnchor? = nil,
        padding: UIEdgeInsets = .zero,
        size: CGSize = .zero
    ) {
        translatesAutoresizingMaskIntoConstraints = false
        
        if let top = top {
            topAnchor.constraint(equalTo: top, constant: padding.top).isActive = true
        }
        if let leading = leading {
            leadingAnchor.constraint(equalTo: leading, constant: padding.left).isActive = true
        }
        if let bottom = bottom {
            bottomAnchor.constraint(equalTo: bottom, constant: -padding.bottom).isActive = true
        }
        if let trailing = trailing {
            trailingAnchor.constraint(equalTo: trailing, constant: -padding.right).isActive = true
        }
        if size.width != 0 {
            widthAnchor.constraint(equalToConstant: size.width).isActive = true
        }
        if size.height != 0 {
            heightAnchor.constraint(equalToConstant: size.height).isActive = true
        }
    }
}

// Использование
myView.anchor(
    top: view.safeAreaLayoutGuide.topAnchor,
    leading: view.leadingAnchor,
    trailing: view.trailingAnchor,
    padding: UIEdgeInsets(top: 20, left: 16, bottom: 0, right: 16),
    size: CGSize(width: 0, height: 100)
)
```

---

### Итог

**NSLayoutAnchor не заменяет Auto Layout**, а является современным, типобезопасным и читаемым API для создания констрейнтов.

| Аспект                  | Классический API     | NSLayoutAnchor             |
| ----------------------- | -------------------- | -------------------------- |
| **Синтаксис**           | Громоздкий           | Лаконичный                 |
| **Безопасность**        | Ошибки в [[Runtime]] | Ошибки на этапе компиляции |
| **Читаемость**          | Низкая               | Высокая                    |
| **Скорость разработки** | Низкая               | Высокая                    |
| **Рекомендация Apple**  | ❌                    | ✅                          |

**Вывод:** В современных iOS-проектах (iOS 9+) используйте только **NSLayoutAnchor**. Классический API оставьте для поддержки легаси-кода или очень специфических задач, где требуется прямой доступ к `NSLayoutConstraint`.