## 1. Что такое `UIBarItem`

**`UIBarItem`** — это **базовый класс** для элементов, которые могут отображаться **на панелях (bar)**:

- **[[UITabBar]]** (вкладки внизу экрана)
    
- **[[UIToolbar]]** (панель с кнопками внизу/сверху)
    
- **[[UINavigationBar]]** (кнопки слева/справа или заголовок)
    

`UIBarItem` сам по себе не используется напрямую, а служит **родительским классом** для:

- `UIBarButtonItem`
    
- `UITabBarItem`
    

---

## 2. Основные свойства `UIBarItem`

|Свойство|Описание|
|---|---|
|**title**|Заголовок элемента|
|**image**|Изображение элемента|
|**selectedImage**|Изображение при выбранном состоянии (например, у вкладок)|
|**badgeValue**|Маленький кружок с числом (например, уведомления в таббаре)|
|**tag**|Целое число для идентификации элемента|
|**isEnabled**|Доступен элемент или нет|

---

## 3. Примеры

### Пример 1. [[UITabBarItem]]

```swift
let tabBarItem = UITabBarItem(
    title: "Главная",
    image: UIImage(systemName: "house"),
    selectedImage: UIImage(systemName: "house.fill")
)

viewController.tabBarItem = tabBarItem
```

👉 Теперь контроллер будет отображаться во вкладке с иконкой и заголовком.

---

### Пример 2. [[UIBarButtonItem]]

```swift
let barButton = UIBarButtonItem(
    image: UIImage(systemName: "gear"),
    style: .plain,
    target: self,
    action: #selector(openSettings)
)

navigationItem.rightBarButtonItem = barButton

@objc func openSettings() {
    print("Настройки нажаты")
}
```

👉 Кнопка с иконкой шестерёнки добавлена в `UINavigationBar` справа.

---

### Пример 3. Badge (счётчик)

```swift
tabBarItem.badgeValue = "5"
```

👉 Покажет красный кружок с числом `5` на иконке вкладки.

---

## 4. Особенности

- `UIBarItem` **не является [[Swift/Теория/UIKit/UIView]]**  
    → это объект, описывающий элемент интерфейса (иконку, текст, состояние).
    
- Его визуальное отображение зависит от того, где он используется (TabBar, Toolbar, NavigationBar).
    
- Через `tag` удобно отличать элементы, если у них один обработчик.
    

---

## 5. Итог

- **UIBarItem** = базовый элемент для кнопок и вкладок в панелях.
    
- Дочерние классы: `UIBarButtonItem`, `UITabBarItem`.
    
- Может иметь **иконку, текст, бейдж, доступность**.
    
- Управляет состоянием элемента, но сам **не рисует UI**, это делает [[UIKit]].
    

---
