## 1. Что такое `UIToolbar`

**`UIToolbar`** — это **панель инструментов**, которая обычно располагается внизу экрана и содержит:

- Кнопки ([[UIBarButtonItem]])
    
- Пространство для выравнивания (`flexibleSpace`, `fixedSpace`)
    
- Пользовательские элементы (`customView`)
    

Особенности:

- Может содержать **несколько элементов**
    
- Поддерживает **анимацию скрытия и отображения**
    
- Может использоваться как в **[[UIViewController]]**, так и в **[[UINavigationController]]**
    

> Проще говоря: `UIToolbar` = «панель кнопок внизу экрана».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**items**|Массив элементов (UIBarButtonItem) на панели|
|**barStyle**|Стиль панели (`default`, `black`)|
|**isTranslucent**|Если true, панель прозрачная|
|**sizeToFit()**|Автоматическая подгонка размера панели под экран|
|**UIBarButtonItem.flexibleSpace**|Гибкое пространство между кнопками|
|**UIBarButtonItem.fixedSpace**|Фиксированное пространство между кнопками|

---

## 3. Основной синтаксис

### Создание UIToolbar программно

```swift
let toolbar = UIToolbar(frame: CGRect(x: 0, y: 700, width: view.frame.width, height: 50))
toolbar.barStyle = .default
toolbar.isTranslucent = true
toolbar.sizeToFit()

let cancelButton = UIBarButtonItem(barButtonSystemItem: .cancel, target: self, action: #selector(cancelTapped))
let flexibleSpace = UIBarButtonItem(barButtonSystemItem: .flexibleSpace, target: nil, action: nil)
let doneButton = UIBarButtonItem(barButtonSystemItem: .done, target: self, action: #selector(doneTapped))

toolbar.setItems([cancelButton, flexibleSpace, doneButton], animated: false)
view.addSubview(toolbar)

@objc func cancelTapped() { print("Cancel tapped") }
@objc func doneTapped() { print("Done tapped") }
```

---

## 4. Примеры от простого к сложному

### Пример 1. Простейшая панель с одной кнопкой

```swift
let toolbar = UIToolbar()
let doneButton = UIBarButtonItem(barButtonSystemItem: .done, target: self, action: #selector(doneTapped))
toolbar.setItems([doneButton], animated: false)
view.addSubview(toolbar)
```

- Панель с одной кнопкой Done
    

---

### Пример 2. Несколько кнопок с гибким пространством

```swift
let addButton = UIBarButtonItem(barButtonSystemItem: .add, target: self, action: #selector(addTapped))
let editButton = UIBarButtonItem(barButtonSystemItem: .edit, target: self, action: #selector(editTapped))
let flexibleSpace = UIBarButtonItem(barButtonSystemItem: .flexibleSpace, target: nil, action: nil)

toolbar.setItems([addButton, flexibleSpace, editButton], animated: true)
```

- Кнопки выровнены с помощью гибкого пространства
    

---

### Пример 3. Панель с пользовательским элементом

```swift
let customLabel = UILabel()
customLabel.text = "Toolbar Label"
let customItem = UIBarButtonItem(customView: customLabel)

toolbar.setItems([customItem], animated: false)
```

- Добавляем [[UILabel]] вместо стандартной кнопки
    

---

### Пример 4. Прозрачная панель

```swift
toolbar.isTranslucent = true
toolbar.barTintColor = UIColor.clear
toolbar.backgroundColor = UIColor.clear
```

- Панель выглядит полупрозрачной или полностью прозрачной
    

---

### Пример 5. Анимация появления и скрытия

```swift
UIView.animate(withDuration: 0.3) {
    toolbar.alpha = 0 // скрыть
}

UIView.animate(withDuration: 0.3) {
    toolbar.alpha = 1 // показать
}
```

- Анимируем показ и скрытие панели инструментов
    

---

## 5. Особенности UIToolbar

1. Поддерживает **только UIBarButtonItem** или пользовательские элементы
    
2. Можно использовать **flexibleSpace** для выравнивания кнопок
    
3. Часто используется вместе с **[[UITextField]]** или **[[UITextView]]** для добавления кнопок Done/Cancel над клавиатурой
    
4. Отличие от UINavigationBar: **UIToolbar** — внизу экрана, а [[UINavigationBar]] — сверху
    

---

## 6. Итог

- **UIToolbar** = панель инструментов с кнопками или кастомными элементами
    
- Позволяет:
    
    - Добавлять системные кнопки или customView
        
    - Выравнивать кнопки с помощью flexible/fixed space
        
    - Настраивать прозрачность и стиль
        
    - Анимировать появление и скрытие
        

---
