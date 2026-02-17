**`UIViewController`** — это объект, который управляет **одним экраном или его частью** в [[iOS]]-приложении.

🔑 Основные обязанности:

1. Управление **view** — основной иерархией элементов на экране.
    
2. Обработка **жизненного цикла**: загрузка, отображение, исчезновение экрана.
    
3. Реакция на события пользователя.
    
4. Координация **переходов между экранами**.
    

> Проще: `UIViewController` = логика + экран.

---

## 2. Основные свойства

| Свойство                   | Описание                                                   |
| -------------------------- | ---------------------------------------------------------- |
| `view`                     | Главная view контроллера (типа [[Swift/Теория/UIKit/UIView]])                 |
| `navigationItem`           | Настройки навигации (заголовок, кнопки)                    |
| `tabBarItem`               | Элемент таб-бара, если контроллер в [[UITabBarController]] |
| `parent`                   | Родительский контроллер (если вложен)                      |
| `children`                 | Дочерние контроллеры (для контейнеров)                     |
| `presentedViewController`  | Контроллер, который был показан поверх текущего            |
| `presentingViewController` | Контроллер, который вызвал текущий через `present`         |

---

## 3. Жизненный цикл UIViewController

Основные методы:

| Метод                                    | Когда вызывается                                              |
| ---------------------------------------- | ------------------------------------------------------------- |
| `init(nibName:bundle:)` / `init(coder:)` | Инициализация контроллера                                     |
| `loadView()`                             | Создание view, если не загружено из XIB/Storyboard            |
| `viewDidLoad()`                          | После загрузки view в память (вызывается один раз)            |
| `viewWillAppear(_:)`                     | Перед тем, как view появится на экране                        |
| `updateViewConstraints()`                | Перед layout, если есть кастомные constraints                 |
| `viewWillLayoutSubviews()`               | Перед раскладкой субвью                                       |
| `viewDidLayoutSubviews()`                | После раскладки субвью                                        |
| `viewDidAppear(_:)`                      | После того, как view появилось на экране                      |
| `viewWillDisappear(_:)`                  | Перед тем, как view исчезнет с экрана                         |
| `viewDidDisappear(_:)`                   | После того, как view исчезло с экрана                         |
| `viewWillTransition(to:with:)`           | Перед изменением размера view (например, при повороте экрана) |
| `traitCollectionDidChange(_:)`           | При изменении size class или темной/светлой темы              |
| `didReceiveMemoryWarning()`              | Когда система сообщает о нехватке памяти                      |
| `deinit`                                 | Когда контроллер уничтожается из памяти                       |
| `viewDidLoad()`                          | После загрузки view в память (один раз)                       |
| `viewWillAppear(_:)`                     | Перед тем, как view появится на экране                        |
| `viewDidAppear(_:)`                      | После того, как view появилось                                |
| `viewWillDisappear(_:)`                  | Перед тем, как view исчезнет                                  |
| `viewDidDisappear(_:)`                   | После того, как view исчезло                                  |
| `deinit`                                 | Когда контроллер уничтожается из памяти                       |

---

## 4. Пример кода

### Простейший `UIViewController`

```swift
class HomeViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
        
        let label = UILabel(frame: CGRect(x: 50, y: 100, width: 200, height: 50))
        label.text = "Главная"
        label.textColor = .black
        view.addSubview(label)
    }
}
```

---

### Переход на другой экран

```swift
let detailsVC = DetailsViewController()
navigationController?.pushViewController(detailsVC, animated: true)

// или модально
present(detailsVC, animated: true)
```

---

### Настройка заголовка и кнопок навигации

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    navigationItem.title = "Главная"
    navigationItem.rightBarButtonItem = UIBarButtonItem(
        title: "Настройки",
        style: .plain,
        target: self,
        action: #selector(openSettings)
    )
}

@objc func openSettings() {
    print("Настройки открыты")
}
```

---

### Добавление дочернего контроллера

```swift
let childVC = ChildViewController()
addChild(childVC)
childVC.view.frame = CGRect(x: 0, y: 200, width: view.frame.width, height: 200)
view.addSubview(childVC.view)
childVC.didMove(toParent: self)
```

---

## 5. Особенности

1. **Контролирует один экран**, но может содержать дочерние контроллеры.
    
2. **Не наследует `UIView`**, но управляет ним через свойство `view`.
    
3. Используется вместе с **[[UINavigationController]], [[UITabBarController]], [[UISplitViewController]]** для построения интерфейса.
    
4. Поддерживает **жизненный цикл**, который важно знать для инициализации и очистки ресурсов.
    

---

## 6. Итог

- `UIViewController` = **экран + логика**.
    
- Обязанности: управление view, обработка событий, навигация.
    
- Важная часть [[UIKit]], без него нельзя строить экранные интерфейсы.
    
- Хорошо комбинируется с контейнерами: навигация и таб-бары.
    

---
