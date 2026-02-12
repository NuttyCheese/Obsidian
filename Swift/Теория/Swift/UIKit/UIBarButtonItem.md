## 1. Что такое `UIBarButtonItem`

**`UIBarButtonItem`** — это **кнопка или элемент управления**, предназначенный для размещения в:

- **[[UINavigationBar]]** (верхняя панель навигации)
    
- **[[UIToolbar]]** (панель инструментов внизу экрана)
    

Особенности:

- Не является **[[Swift/Теория/Swift/UIKit/UIView]]**, а **[[UIBarItem]]**
    
- Может отображать:
    
    - Текст (`title`)
        
    - Изображение (`image`)
        
    - Системные кнопки (`.add`, `.edit`, `.done` и др.)
        
- Может иметь **действие (`action`)** через target-action
    

> Проще говоря: `UIBarButtonItem` = кнопка для навигационной панели или тулбара.

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**title**|Текст кнопки|
|**image**|Изображение для кнопки|
|**style**|Стиль кнопки (`.plain`, `.done`, `.bordered`)|
|**target-action**|Механизм, который вызывается при нажатии кнопки|
|**systemItem**|Предустановленные системные кнопки (`.add`, `.edit`, `.save`, и др.)|
|**UINavigationItem.leftBarButtonItem / rightBarButtonItem**|Размещение кнопки слева или справа в NavigationBar|
|**UIToolbar.items**|Размещение кнопок в тулбаре|

---

## 3. Основной синтаксис

### Создание кнопки с текстом

```swift
let button = UIBarButtonItem(title: "Save", style: .plain, target: self, action: #selector(saveTapped))
```

### Создание кнопки с изображением

```swift
let button = UIBarButtonItem(image: UIImage(systemName: "plus"), style: .plain, target: self, action: #selector(addTapped))
```

### Создание системной кнопки

```swift
let button = UIBarButtonItem(barButtonSystemItem: .edit, target: self, action: #selector(editTapped))
```

---

## 4. Примеры от простого к сложному

### Пример 1. Простая текстовая кнопка

```swift
let saveButton = UIBarButtonItem(title: "Save", style: .plain, target: self, action: #selector(saveTapped))
navigationItem.rightBarButtonItem = saveButton

@objc func saveTapped() {
    print("Save button tapped")
}
```

- Кнопка справа в NavigationBar с текстом
    

---

### Пример 2. Кнопка с изображением

```swift
let addButton = UIBarButtonItem(image: UIImage(systemName: "plus"), style: .plain, target: self, action: #selector(addTapped))
navigationItem.leftBarButtonItem = addButton

@objc func addTapped() {
    print("Add button tapped")
}
```

- Кнопка слева с иконкой из SF Symbols
    

---

### Пример 3. Системная кнопка

```swift
let editButton = UIBarButtonItem(barButtonSystemItem: .edit, target: self, action: #selector(editTapped))
navigationItem.rightBarButtonItem = editButton

@objc func editTapped() {
    print("Edit button tapped")
}
```

- Используем готовый системный стиль `.edit`
    

---

### Пример 4. Несколько кнопок в NavigationBar

```swift
let addButton = UIBarButtonItem(barButtonSystemItem: .add, target: self, action: #selector(addTapped))
let editButton = UIBarButtonItem(barButtonSystemItem: .edit, target: self, action: #selector(editTapped))
navigationItem.rightBarButtonItems = [addButton, editButton]

@objc func addTapped() { print("Add tapped") }
@objc func editTapped() { print("Edit tapped") }
```

- Размещаем несколько кнопок справа
    

---

### Пример 5. UIBarButtonItem в UIToolbar

```swift
let toolbar = UIToolbar(frame: CGRect(x: 0, y: 700, width: view.frame.width, height: 50))

let doneButton = UIBarButtonItem(barButtonSystemItem: .done, target: self, action: #selector(doneTapped))
let flexibleSpace = UIBarButtonItem(barButtonSystemItem: .flexibleSpace, target: nil, action: nil)
let cancelButton = UIBarButtonItem(barButtonSystemItem: .cancel, target: self, action: #selector(cancelTapped))

toolbar.setItems([cancelButton, flexibleSpace, doneButton], animated: false)
view.addSubview(toolbar)

@objc func doneTapped() { print("Done tapped") }
@objc func cancelTapped() { print("Cancel tapped") }
```

- Используем тулбар с кнопками и гибким пространством
    

---

## 5. Особенности UIBarButtonItem

1. **Не является UIView**, поэтому нельзя напрямую изменять frame
    
2. Для кастомного UI используют **customView**
    
3. Поддерживает как текст, так и изображение
    
4. Может быть **несколько кнопок** в NavigationBar и Toolbar
    
5. Использует **target-action** для обработки нажатий
    

---

## 6. Итог

- **UIBarButtonItem** = кнопка для NavigationBar или UIToolbar
    
- Позволяет:
    
    - Добавлять текстовые, графические и системные кнопки
        
    - Использовать несколько кнопок сразу
        
    - Привязывать действия через target-action
        
- Отличие от UIButton: **не UIView**, управляется через UIBarButtonItem API
    

---
