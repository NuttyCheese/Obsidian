## 1. Что такое `UIDatePicker`

**`UIDatePicker`** — это **[[UIControl]]**, который позволяет пользователю выбрать:

- дату ([[Date]])
    
- время ([[Time]])
    
- дату и время одновременно
    
- диапазон или календарь ([[iOS]] 14+)
    

Он заменяет ручной ввод даты и обеспечивает единообразие интерфейса.

---

## 2. Инициализация и базовая настройка

```swift
let datePicker = UIDatePicker()
datePicker.datePickerMode = .dateAndTime  // режим: дата, время, дата+время
datePicker.locale = Locale(identifier: "ru_RU") // локализация
datePicker.preferredDatePickerStyle = .wheels // стиль колёсиков (iOS 14+)
```

---

## 3. Настройка диапазона и начальной даты

```swift
datePicker.date = Date() // текущая дата
datePicker.minimumDate = Calendar.current.date(byAdding: .year, value: -1, to: Date())
datePicker.maximumDate = Calendar.current.date(byAdding: .year, value: 1, to: Date())
```

---

## 4. Событие изменения значения

Для отслеживания изменения даты используют событие **[[ValueChanged]]**:

```swift
datePicker.addTarget(self, action: #selector(dateChanged(_:)), for: .valueChanged)

@objc func dateChanged(_ sender: UIDatePicker) {
    let formatter = DateFormatter()
    formatter.dateStyle = .medium
    formatter.timeStyle = .short
    print("Выбранная дата: \(formatter.string(from: sender.date))")
}
```

---

## 5. Пример использования в `UIViewController`

```swift
class DatePickerViewController: UIViewController {
    let datePicker = UIDatePicker()

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white

        datePicker.datePickerMode = .date
        datePicker.preferredDatePickerStyle = .wheels
        datePicker.addTarget(self, action: #selector(dateChanged(_:)), for: .valueChanged)

        datePicker.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(datePicker)

        NSLayoutConstraint.activate([
            datePicker.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            datePicker.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }

    @objc func dateChanged(_ sender: UIDatePicker) {
        print("Выбрана дата: \(sender.date)")
    }
}
```

---

## 6. Основные свойства UIDatePicker

|Свойство|Описание|
|---|---|
|`datePickerMode`|Режим: `.date`, `.time`, `.dateAndTime`, `.countDownTimer`|
|`date`|Текущая выбранная дата|
|`minimumDate` / `maximumDate`|Ограничения по дате|
|`locale`|Локализация отображения|
|`minuteInterval`|Интервал минут (например, шаг 5 минут)|
|`preferredDatePickerStyle`|Стиль отображения (`wheels`, `compact`, `inline`)|

---

## 7. Итог

- `UIDatePicker` = удобный элемент для выбора даты и времени.
    
- Событие `.valueChanged` позволяет получать выбранное значение сразу.
    
- Можно кастомизировать стиль, локаль, диапазон, шаг минут.
    
- Подходит для форм, планировщиков, напоминаний и т.д.
    

---

