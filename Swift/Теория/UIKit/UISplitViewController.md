## 1. Что такое `UISplitViewController`

**`UISplitViewController`** — это **контейнерный контроллер**, который управляет **двумя и более контроллерами**:

- **Primary (основной/мастер)** — список, меню, навигация.
    
- **Secondary (подчинённый/detail)** — детали выбранного элемента.
    

> Обычно используется для iPad, но с [[iOS]] 14+ есть адаптивный режим для iPhone.

---

## 2. Основные особенности

- Автоматическая адаптация интерфейса под размер экрана.
    
- Поддерживает **stack-навигацию** внутри колонок.
    
- Можно настраивать **колонки**:
    
    - `.primary` — левая (или верхняя на iPhone)
        
    - `.secondary` — правая (или деталь под основным на iPhone)
        
    - `.supplementary` — дополнительная колонка (iOS 14+)
        
- Использует **контейнерный паттерн**, поэтому `viewControllers` — массив контроллеров.
    

---

## 3. Пример простого SplitViewController

```swift
let masterVC = MasterViewController() // список
let detailVC = DetailViewController() // детали

let splitVC = UISplitViewController(style: .doubleColumn)
splitVC.setViewController(masterVC, for: .primary)
splitVC.setViewController(detailVC, for: .secondary)

window?.rootViewController = splitVC
window?.makeKeyAndVisible()
```

---

## 4. Настройка колонок

- `primaryBackgroundStyle` — фон основной колонки
    
- `preferredDisplayMode` — как показывать колонки:
    

```swift
splitVC.preferredDisplayMode = .oneBesideSecondary // две колонки рядом
splitVC.preferredSplitBehavior = .tile // iPad: равномерное распределение
```

- Можно менять ширину колонок:
    

```swift
splitVC.maximumPrimaryColumnWidth = 300
splitVC.minimumPrimaryColumnWidth = 200
```

---

## 5. Динамическое управление контроллерами

```swift
let extraVC = ExtraViewController()
splitVC.setViewController(extraVC, for: .supplementary) // iOS 14+
```

- Можно обновлять контроллеры во время работы:
    

```swift
splitVC.showDetailViewController(newDetailVC, sender: self)
```

---

## 6. Жизненный цикл и делегат

- [[UISplitViewControllerDelegate]] позволяет:
    
    - контролировать поведение колонок,
        
    - реагировать на скрытие/показ колонок,
        
    - управлять адаптацией под разные размеры экрана.
        

Пример делегата:

```swift
extension MainSplitViewController: UISplitViewControllerDelegate {
    func splitViewController(
        _ svc: UISplitViewController,
        collapseSecondary secondaryViewController: UIViewController,
        onto primaryViewController: UIViewController
    ) -> Bool {
        // возвращаем true, если хотим полностью контролировать collapse
        return false
    }
}
```

---

## 7. Итог

- `UISplitViewController` = адаптивный контейнер с колонками.
    
- Основные сценарии: Master-Detail интерфейс, особенно на iPad.
    
- Поддерживает до 3 колонок (`primary`, `secondary`, `supplementary`).
    
- Комбинируется с [[UINavigationController]] внутри каждой колонки.
    
- Делегат позволяет гибко контролировать поведение при изменении размера экрана.
    

---
