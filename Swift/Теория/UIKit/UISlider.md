**`UISlider`** — это **ползунок**, который позволяет пользователю выбрать **числовое значение из диапазона**.

- Отображает **ползунок на треке**
    
- Можно задать:
    
    - Минимальное и максимальное значение (`minimumValue`, `maximumValue`)
        
    - Начальное значение (`value`)
        
    - Цвет трека и ползунка
        
- Поддерживает **события изменения значения** (`.valueChanged`)
    

> Проще говоря: `UISlider` = «ползунок для выбора числа».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**value**|Текущее значение ползунка|
|**minimumValue**|Минимальное значение|
|**maximumValue**|Максимальное значение|
|**isContinuous**|Если true, событие `.valueChanged` вызывается непрерывно при движении ползунка|
|**minimumTrackTintColor**|Цвет левой (нижней) части трека|
|**maximumTrackTintColor**|Цвет правой (верхней) части трека|
|**thumbTintColor**|Цвет ползунка (кнопки)|
|**addTarget**|Привязка действия к событию ползунка|

---

## 3. Основной синтаксис

### Создание ползунка программно

```swift
let slider = UISlider(frame: CGRect(x: 50, y: 200, width: 300, height: 30))
slider.minimumValue = 0
slider.maximumValue = 100
slider.value = 50
slider.addTarget(self, action: #selector(sliderChanged(_:)), for: .valueChanged)
view.addSubview(slider)

@objc func sliderChanged(_ sender: UISlider) {
    print("Slider value: \(sender.value)")
}
```

---

## 4. Примеры от простого к сложному

### Пример 1. Простейший ползунок

```swift
let slider = UISlider()
slider.minimumValue = 0
slider.maximumValue = 10
slider.value = 5
view.addSubview(slider)
```

- Простой ползунок без действий
    

---

### Пример 2. Отслеживание изменения значения

```swift
slider.addTarget(self, action: #selector(sliderChanged(_:)), for: .valueChanged)

@objc func sliderChanged(_ sender: UISlider) {
    print("Текущее значение: \(sender.value)")
}
```

- Вывод значения в консоль при каждом изменении
    

---

### Пример 3. Настройка цветов

```swift
slider.minimumTrackTintColor = .systemBlue
slider.maximumTrackTintColor = .systemGray
slider.thumbTintColor = .systemRed
```

- Левый трек синий, правый серый, ползунок красный
    

---

### Пример 4. Целочисленные значения

```swift
@objc func sliderChanged(_ sender: UISlider) {
    let roundedValue = round(sender.value)
    print("Целое значение: \(Int(roundedValue))")
}
```

- Округляем до целого числа, если нужен шаг в 1
    

---

### Пример 5. Ползунок с UI обновлением

```swift
let valueLabel = UILabel(frame: CGRect(x: 50, y: 250, width: 100, height: 30))
valueLabel.text = "50"
view.addSubview(valueLabel)

@objc func sliderChanged(_ sender: UISlider) {
    let rounded = Int(sender.value)
    valueLabel.text = "\(rounded)"
}
```

- Ползунок управляет значением [[UILabel]]
    

---

## 5. Особенности UISlider

1. Значение хранится как **Float**
    
2. Можно настроить **диапазон, цвета и непрерывность**
    
3. Поддерживает **события [[ValueChanged]]** для интерактивного обновления
    
4. Для дискретных шагов нужно вручную округлять или использовать `minimumValue` и `maximumValue` с шагом
    
5. Часто используется для:
    
    - Настройки громкости
        
    - Яркости экрана
        
    - Выбора параметров (скорость, размер, интенсивность)
        

---

## 6. Итог

- **UISlider** = ползунок для выбора значения из диапазона
    
- Позволяет:
    
    - Настраивать диапазон и начальное значение
        
    - Менять цвета трека и ползунка
        
    - Отслеживать изменения значения и обновлять UI
        

---
