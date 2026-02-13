## 1. Что такое `CALayer`

**`CALayer`** — это объект, который управляет **визуальным содержимым и анимациями элементов интерфейса**.

- Каждое [[Swift/Теория/UIKit/UIView]] содержит **свой слой `layer`**, через который можно управлять:
    
    - **Границами** (`borderWidth`, `borderColor`)
        
    - **Скруглениями углов** (`cornerRadius`)
        
    - **Тенью** (`shadowOpacity`, `shadowOffset`, `shadowColor`)
        
    - **Анимациями** (CABasicAnimation, CAKeyframeAnimation)
        
- `CALayer` может содержать **подслои (`sublayers`)**, создавая иерархию визуальных слоев
    

> Проще говоря: `CALayer` = «невидимый слой под UIView, который управляет всеми визуальными эффектами и анимациями».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**layer**|Свойство UIView, возвращает CALayer, связанный с этим UIView|
|**sublayers**|Массив подслоев, которые находятся внутри слоя|
|**cornerRadius**|Скругление углов слоя|
|**shadowOpacity / shadowColor / shadowOffset**|Свойства тени слоя|
|**borderWidth / borderColor**|Граница слоя|
|**mask / contents**|Маска слоя или содержимое (изображение, цвет)|
|**CABasicAnimation / CAKeyframeAnimation**|Анимации, которые применяются к CALayer|

---

## 3. Основной синтаксис

```swift
let view = UIView(frame: CGRect(x: 50, y: 50, width: 100, height: 100))
view.backgroundColor = .systemBlue

// Работа с layer
view.layer.cornerRadius = 10
view.layer.borderWidth = 2
view.layer.borderColor = UIColor.black.cgColor
view.layer.shadowColor = UIColor.black.cgColor
view.layer.shadowOpacity = 0.5
view.layer.shadowOffset = CGSize(width: 3, height: 3)
```

- `view.layer` возвращает связанный слой для любого UIView
    
- Многие свойства слоя используют **CGColor** вместо UIColor
    

---

## 4. Примеры от простого к сложному

### Пример 1. Скругление углов

```swift
let view = UIView(frame: CGRect(x: 50, y: 50, width: 100, height: 100))
view.backgroundColor = .systemRed
view.layer.cornerRadius = 20
```

- Простейшее визуальное изменение слоя
    

---

### Пример 2. Добавление тени

```swift
view.layer.shadowColor = UIColor.black.cgColor
view.layer.shadowOpacity = 0.7
view.layer.shadowOffset = CGSize(width: 5, height: 5)
view.layer.shadowRadius = 10
```

- Слой создаёт **тень вокруг UIView**
    

---

### Пример 3. Граница слоя

```swift
view.layer.borderWidth = 2
view.layer.borderColor = UIColor.white.cgColor
```

- Добавляет **рамку вокруг UIView**
    

---

### Пример 4. Добавление подслоя

```swift
let subLayer = CALayer()
subLayer.frame = CGRect(x: 10, y: 10, width: 80, height: 80)
subLayer.backgroundColor = UIColor.yellow.cgColor
view.layer.addSublayer(subLayer)
```

- Создание и добавление **подслоя** в существующий слой
    

---

### Пример 5. Анимация слоя

```swift
let animation = CABasicAnimation(keyPath: "opacity")
animation.fromValue = 0
animation.toValue = 1
animation.duration = 1.0
view.layer.add(animation, forKey: "fadeIn")
```

- Простейшая анимация появления слоя
    

---

## 5. Особенности CALayer

1. **Каждый UIView имеет слой** → view.layer
    
2. Свойства слоя используют **Core Graphics типы** (CGColor, CGSize, CGPoint)
    
3. Позволяет **создавать сложные визуальные эффекты и анимации**
    
4. Поддерживает **иерархию слоев через sublayers**
    
5. Можно применять анимации через **CABasicAnimation, CAKeyframeAnimation, CATransaction**
    

---

## 6. Итог

- **CALayer** = невидимый слой, управляющий визуальным представлением UIView
    
- Позволяет:
    
    - Скруглять углы, добавлять тень, рамку
        
    - Создавать подслои
        
    - Анимировать свойства слоя
        
    - Использовать Core Animation для плавных эффектов
        
- Слой отделён от логики UIView, что делает анимации и визуальные эффекты эффективнее
    

---
