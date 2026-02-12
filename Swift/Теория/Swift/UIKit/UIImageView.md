## 1. Что такое `UIImageView`

**`UIImageView`** — это подкласс **[[Swift/Теория/Swift/UIKit/UIView]]**, предназначенный для отображения **изображений ([[UIImage]])** в приложении [[iOS]].

- Не интерактивен по умолчанию (не реагирует на нажатия)
    
- Может содержать одно изображение и анимированную последовательность изображений
    
- Поддерживает **contentMode**, чтобы управлять тем, как изображение масштабируется и размещается внутри view
    

> Проще говоря: `UIImageView` = «виджет для показа картинки на экране».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**UIImage**|Объект изображения, который отображается в UIImageView|
|**contentMode**|Определяет, как изображение размещается и масштабируется (`.scaleAspectFit`, `.scaleAspectFill` и др.)|
|**isUserInteractionEnabled**|Разрешает взаимодействие с изображением, например для распознавания тапов|
|**animationImages**|Массив изображений для анимации|
|**startAnimating / stopAnimating**|Методы запуска и остановки анимации изображений|
|**clipsToBounds**|Обрезает изображение по границам view, если оно выходит за пределы|

---

## 3. Основной синтаксис

### Через Interface Builder

```swift
@IBOutlet weak var imageView: UIImageView!
imageView.image = UIImage(named: "exampleImage")
imageView.contentMode = .scaleAspectFit
```

### Программно

```swift
let imageView = UIImageView(frame: CGRect(x: 50, y: 50, width: 200, height: 200))
imageView.image = UIImage(named: "exampleImage")
imageView.contentMode = .scaleAspectFill
imageView.clipsToBounds = true
view.addSubview(imageView)
```

---

## 4. Примеры от простого к сложному

### Пример 1. Простое изображение

```swift
let imageView = UIImageView()
imageView.image = UIImage(named: "exampleImage")
imageView.frame = CGRect(x: 50, y: 50, width: 100, height: 100)
view.addSubview(imageView)
```

- Просто отображаем картинку на экране
    

---

### Пример 2. Content Mode

```swift
imageView.contentMode = .scaleAspectFit // Вписывает изображение внутри view, сохраняя пропорции
imageView.contentMode = .scaleAspectFill // Масштабирует изображение, чтобы заполнить весь view
```

- Управляем тем, как изображение отображается внутри UIImageView
    

---

### Пример 3. Анимация изображений

```swift
let animationImages = [
    UIImage(named: "frame1")!,
    UIImage(named: "frame2")!,
    UIImage(named: "frame3")!
]
imageView.animationImages = animationImages
imageView.animationDuration = 1.0
imageView.startAnimating()
```

- UIImageView может показывать последовательность изображений как анимацию
    

---

### Пример 4. Скругление и тень через layer

```swift
imageView.layer.cornerRadius = 20
imageView.clipsToBounds = true
imageView.layer.shadowColor = UIColor.black.cgColor
imageView.layer.shadowOpacity = 0.5
imageView.layer.shadowOffset = CGSize(width: 3, height: 3)
```

- Можно комбинировать **[[CALayer]]** свойства с UIImageView
    

---

### Пример 5. UIImageView с распознаванием тапов

```swift
imageView.isUserInteractionEnabled = true
let tapGesture = UITapGestureRecognizer(target: self, action: #selector(imageTapped))
imageView.addGestureRecognizer(tapGesture)

@objc func imageTapped() {
    print("Image tapped!")
}
```

- По умолчанию UIImageView **не реагирует на касания**, нужно включить `isUserInteractionEnabled` и добавить жест
    

---

## 5. Особенности UIImageView

1. **Не интерактивен по умолчанию**
    
2. Поддерживает как **одно изображение, так и анимацию**
    
3. Использует **contentMode** для управления масштабированием
    
4. Можно комбинировать с **слоями (`CALayer`)** для скруглений, теней и рамок
    
5. Для взаимодействия нужно включать `isUserInteractionEnabled` и использовать жесты
    

---

## 6. Итог

- **UIImageView** = компонент для отображения изображений
    
- Позволяет:
    
    - Показывать статические картинки
        
    - Создавать анимацию через `animationImages`
        
    - Настраивать контент (масштаб, обрезку)
        
    - Использовать CALayer для скруглений, тени и границ
        
    - Обрабатывать нажатия через [[UITapGestureRecognizer]]
        

---
