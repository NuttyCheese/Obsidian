## 1. Что такое `UIImage`

**`UIImage`** — это класс, который представляет **растровое изображение** или иконку в [[iOS]].

- Может быть загружено из **ресурсов проекта, [[URL]] или данных**
    
- Используется с **[[UIImageView]]**, **[[UIButton]]** и другими элементами UI для отображения изображений
    
- Поддерживает **разные форматы**: PNG, JPEG, GIF (только отдельные кадры, анимацию через UIImageView)
    

> Проще говоря: `UIImage` = «картинка, которую можно показать в приложении».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**UIImageView**|Элемент UI, который отображает UIImage|
|**imageLiteral**|Специальный способ вставки изображения в код через Xcode|
|**init(named:)**|Создание UIImage из ресурсов проекта|
|**init(data:)**|Создание UIImage из данных (Data)|
|**withRenderingMode**|Настройка режима отображения: оригинальный или шаблон (template)|
|**resizableImage**|Позволяет масштабировать изображение с сохранением частей (capInsets)|

---

## 3. Основной синтаксис

### Создание UIImage из ресурсов проекта

```swift
let image = UIImage(named: "exampleImage")
```

- Загружает изображение с именем `exampleImage` из каталога Assets
    

### Создание UIImage из данных

```swift
if let url = URL(string: "https://example.com/image.png"),
   let data = try? Data(contentsOf: url) {
    let image = UIImage(data: data)
}
```

- Загружаем изображение из URL (синхронно)
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простое изображение в UIImageView

```swift
let imageView = UIImageView()
imageView.image = UIImage(named: "exampleImage")
```

- Загружаем картинку и показываем её
    

---

### Пример 2. Изображение в UIButton

```swift
let button = UIButton(type: .system)
button.setImage(UIImage(named: "icon"), for: .normal)
```

- Кнопка с изображением вместо текста
    

---

### Пример 3. Рендеринг режима template

```swift
let image = UIImage(named: "icon")?.withRenderingMode(.alwaysTemplate)
imageView.image = image
imageView.tintColor = .systemRed
```

- Позволяет менять цвет изображения через `tintColor`
    

---

### Пример 4. Масштабируемое изображение с capInsets

```swift
let image = UIImage(named: "bubble")?.resizableImage(
    withCapInsets: UIEdgeInsets(top: 10, left: 10, bottom: 10, right: 10),
    resizingMode: .stretch
)
imageView.image = image
```

- Позволяет масштабировать изображение (например, для чатов с пузырьками), сохраняя края
    

---

### Пример 5. Асинхронная загрузка изображения с URL

```swift
let url = URL(string: "https://example.com/image.png")!
URLSession.shared.dataTask(with: url) { data, response, error in
    guard let data = data, let image = UIImage(data: data) else { return }
    DispatchQueue.main.async {
        imageView.image = image
    }
}.resume()
```

- Загружаем изображение с сети и показываем его в `UIImageView` асинхронно
    

---

## 5. Особенности UIImage

1. **UIImage не отображается сам по себе**, используется вместе с UIImageView или UIButton
    
2. Поддерживает **разные способы создания**: из ресурсов, из данных, из системных символов (SF Symbols)
    
3. Можно использовать **template mode** для динамической смены цвета
    
4. Поддерживает **масштабирование через resizableImage**
    
5. Для сетевых изображений нужно работать **асинхронно**
    

---

## 6. Итог

- **UIImage** = объект изображения в iOS
    
- Позволяет:
    
    - Загружать изображения из ресурсов, данных или сети
        
    - Использовать с UIImageView, UIButton, [[UIBarButtonItem]] и др.
        
    - Масштабировать, применять цветовые шаблоны и анимацию через UIImageView
        

---
