## 1. Что такое `UIView`

**`UIView`** — это **прямоугольная область на экране**, которая может:

- Отображать контент (цвет, изображения, текст и т.д.)
    
- Служить контейнером для других элементов интерфейса
    
- Обрабатывать взаимодействие пользователя (касания, жесты)
    

Все визуальные элементы [[UIKit]] ([[UILabel]], [[UIButton]], [[UITextField]] и т.д.) **наследуются от UIView**.

> Проще говоря: `UIView` = «прямоугольная область на экране, в которой можно рисовать и размещать элементы».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**frame**|Положение и размер view в системе координат родителя|
|**bounds**|Размер и координаты view относительно самой себя|
|**center**|Центр view относительно родителя|
|**backgroundColor**|Цвет фона view|
|**alpha**|Прозрачность view (0 = полностью прозрачное, 1 = полностью непрозрачное)|
|**isHidden**|Скрыто или видно view|
|**addSubview(_:)**|Добавление дочерней view|
|**removeFromSuperview()**|Удаление view из иерархии|
|**layer**|CALayer для настройки тени, границ, скруглений|
|**clipsToBounds**|Обрезать содержимое, выходящее за границы view|
|**isUserInteractionEnabled**|Можно ли взаимодействовать с view|

---

## 3. Основной синтаксис

### Создание UIView программно

```swift
let myView = UIView(frame: CGRect(x: 50, y: 100, width: 200, height: 150))
myView.backgroundColor = .systemBlue
view.addSubview(myView)
```

---

## 4. Примеры от простого к сложному

### Пример 1. Простейшая view

```swift
let redView = UIView()
redView.frame = CGRect(x: 20, y: 50, width: 100, height: 100)
redView.backgroundColor = .red
view.addSubview(redView)
```

- Создаём красный квадрат на экране
    

---

### Пример 2. Скругленные углы и граница

```swift
redView.layer.cornerRadius = 10
redView.layer.borderWidth = 2
redView.layer.borderColor = UIColor.black.cgColor
```

- Настраиваем внешний вид с помощью [[CALayer]]
    

---

### Пример 3. Тень

```swift
redView.layer.shadowColor = UIColor.black.cgColor
redView.layer.shadowOpacity = 0.5
redView.layer.shadowOffset = CGSize(width: 5, height: 5)
redView.layer.shadowRadius = 5
```

- Добавляем тень под view
    

---

### Пример 4. Добавление подпредставления (Subview)

```swift
let label = UILabel(frame: CGRect(x: 10, y: 10, width: 80, height: 20))
label.text = "Привет"
label.textColor = .white
redView.addSubview(label)
```

- UILabel теперь отображается внутри красного квадрата
    

---

### Пример 5. Анимация

```swift
UIView.animate(withDuration: 1.0) {
    redView.frame.origin.y += 200
    redView.alpha = 0.5
}
```

- Перемещаем view и меняем прозрачность с анимацией
    

---

### Пример 6. Обработка касаний

```swift
redView.isUserInteractionEnabled = true
let tapGesture = UITapGestureRecognizer(target: self, action: #selector(viewTapped))
redView.addGestureRecognizer(tapGesture)

@objc func viewTapped() {
    print("Красная view нажата")
}
```

- Позволяем view реагировать на касания
    

---

## 5. Особенности UIView

1. **Базовый строительный блок интерфейса** — все элементы UI наследуются от UIView
    
2. Поддерживает **иерархию вложенных view**
    
3. Можно настраивать через **layer**: тени, границы, скругления
    
4. Поддерживает **анимации** через UIView.animate
    
5. Может обрабатывать **жесты пользователя**
    
6. Отличие от UIViewController: **UIView — визуальный объект**, [[UIViewController]] — логика и управление представлениями
    

---

## 6. Итог

- **UIView** = прямоугольная область на экране
    
- Позволяет:
    
    - Отображать контент и управлять цветом/прозрачностью
        
    - Складывать дочерние элементы (subview)
        
    - Настраивать стиль (границы, скругления, тени)
        
    - Анимировать изменения и обрабатывать касания
        

---
