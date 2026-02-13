## 1. Что это такое

**`UISplitViewControllerDelegate`** — протокол, который позволяет:

- контролировать поведение колонок (primary, secondary, supplementary),
    
- управлять их отображением при изменении размера экрана,
    
- реагировать на события развёртывания или свертывания деталей (`collapse` и `separate`).
    

> Обычно используется на iPad и iPhone с адаптивным интерфейсом.

---

## 2. Основные методы

### 2.1 Контроль выбора и collapse колонок

```swift
func splitViewController(
    _ svc: UISplitViewController,
    collapseSecondary secondaryViewController: UIViewController,
    onto primaryViewController: UIViewController
) -> Bool
```

- Вызывается, когда `secondary` контроллер нужно свернуть в `primary` (например, на iPhone).
    
- Возвращаем `true`, если хотим полностью контролировать сворачивание, иначе `false`.
    

```swift
func splitViewController(
    _ svc: UISplitViewController,
    separateSecondaryFrom primaryViewController: UIViewController
) -> UIViewController?
```

- Вызывается при разворачивании, чтобы вернуть контроллер для `secondary`.
    

---

### 2.2 Контроль отображения колонок

```swift
optional func primaryViewController(forCollapsing splitViewController: UISplitViewController) -> UIViewController?
```

- Определяет, какой контроллер будет главным при сворачивании.
    

```swift
optional func primaryViewController(forExpanding splitViewController: UISplitViewController) -> UIViewController?
```

- Определяет, какой контроллер станет основным при разворачивании.
    

---

### 2.3 Поддержка кастомной анимации

```swift
optional func splitViewController(
    _ splitViewController: UISplitViewController,
    animationControllerForSeparating separatingViewController: UIViewController,
    from fromViewController: UIViewController
) -> UIViewControllerAnimatedTransitioning?
```

- Позволяет настроить свою анимацию при разделении колонок.
    

---

## 3. Пример использования

```swift
class MainSplitViewController: UISplitViewController, UISplitViewControllerDelegate {

    override func viewDidLoad() {
        super.viewDidLoad()
        self.delegate = self
    }

    // Срабатывает при сворачивании secondary в primary
    func splitViewController(
        _ splitViewController: UISplitViewController,
        collapseSecondary secondaryViewController: UIViewController,
        onto primaryViewController: UIViewController
    ) -> Bool {
        // если secondary пустой, сворачиваем вручную
        if let detailVC = secondaryViewController as? DetailViewController,
           detailVC.item == nil {
            return true
        }
        return false
    }
}
```

---

## 4. Когда использовать

- Контролировать **адаптивный интерфейс** на разных устройствах.
    
- Определять, какой контроллер показывать в **primary/secondary** при изменении ориентации.
    
- Настраивать **кастомные анимации** разворачивания и сворачивания.
    

---

## 5. Итог

- Делегат `UISplitViewControllerDelegate` нужен для гибкой работы с колонками split view.
    
- Основные методы:
    
    - `collapseSecondary` → сворачивание деталей.
        
    - `separateSecondaryFrom` → разворачивание деталей.
        
    - `primaryViewController(forCollapsing/Expanding)` → выбор основного контроллера.
        
- Позволяет адаптировать интерфейс для iPad и iPhone с разной шириной экрана.
    

---
