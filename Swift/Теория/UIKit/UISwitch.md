## 1. Что такое `UISwitch`

**`UISwitch`** — это **переключатель**, который позволяет пользователю выбирать **между двумя состояниями**:

- **ON (включено)**
    
- **OFF (выключено)**
    

Особенности:

- Используется для включения/отключения настроек
    
- Может быть настроен по цвету и анимации
    
- Поддерживает событие [[ValueChanged]]
    

> Проще говоря: `UISwitch` = «ползунок в виде тумблера для выбора да/нет».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**isOn**|Текущее состояние переключателя (`true` = включен, `false` = выключен)|
|**onTintColor**|Цвет переключателя, когда он включен|
|**thumbTintColor**|Цвет кружка-ползунка|
|**tintColor**|Цвет рамки, когда выключен|
|**addTarget**|Привязка действия к событию переключателя|
|**.valueChanged**|Событие, которое вызывается при изменении состояния|

---

## 3. Основной синтаксис

### Создание UISwitch программно

```swift
let mySwitch = UISwitch(frame: CGRect(x: 100, y: 200, width: 0, height: 0))
mySwitch.isOn = true
mySwitch.onTintColor = .systemGreen
mySwitch.thumbTintColor = .white
mySwitch.addTarget(self, action: #selector(switchChanged(_:)), for: .valueChanged)
view.addSubview(mySwitch)

@objc func switchChanged(_ sender: UISwitch) {
    print("Switch is now \(sender.isOn ? "ON" : "OFF")")
}
```

---

## 4. Примеры от простого к сложному

### Пример 1. Простейший UISwitch

```swift
let mySwitch = UISwitch()
mySwitch.isOn = false
view.addSubview(mySwitch)
```

- Просто создаём переключатель без действий
    

---

### Пример 2. Отслеживание изменения состояния

```swift
mySwitch.addTarget(self, action: #selector(switchChanged(_:)), for: .valueChanged)

@objc func switchChanged(_ sender: UISwitch) {
    print("Состояние переключателя: \(sender.isOn)")
}
```

- Логируем состояние при каждом изменении
    

---

### Пример 3. Настройка цветов

```swift
mySwitch.onTintColor = .systemBlue
mySwitch.thumbTintColor = .white
mySwitch.tintColor = .gray
```

- Настраиваем цвета для включенного и выключенного состояния
    

---

### Пример 4. Связь UISwitch с [[UILabel]]

```swift
let statusLabel = UILabel(frame: CGRect(x: 100, y: 250, width: 200, height: 30))
statusLabel.text = "Выключено"
view.addSubview(statusLabel)

@objc func switchChanged(_ sender: UISwitch) {
    statusLabel.text = sender.isOn ? "Включено" : "Выключено"
}
```

- Изменяем текст метки в зависимости от состояния переключателя
    

---

### Пример 5. Анимация включения/выключения

```swift
@objc func switchChanged(_ sender: UISwitch) {
    UIView.animate(withDuration: 0.3) {
        sender.thumbTintColor = sender.isOn ? .systemGreen : .white
        sender.onTintColor = sender.isOn ? .systemGreen : .systemGray
    }
}
```

- Добавляем плавное изменение цветов при переключении
    

---

## 5. Особенности UISwitch

1. Хранит **только два состояния**: `ON` или `OFF`
    
2. **isOn** управляет состоянием
    
3. Поддерживает кастомизацию **цвета и стиля**
    
4. Событие `.valueChanged` используется для реакции на пользовательское взаимодействие
    
5. Часто используется для:
    
    - Настроек
        
    - Переключения режимов (тёмная/светлая тема)
        
    - Включения/выключения функций
        

---

## 6. Итог

- **UISwitch** = двоичный переключатель для включения/выключения
    
- Позволяет:
    
    - Отслеживать состояние
        
    - Настраивать цвета и внешний вид
        
    - Связывать с метками и другими элементами интерфейса
        
- Отличие от UISlider: **только два состояния, а не диапазон значений**
    

---
