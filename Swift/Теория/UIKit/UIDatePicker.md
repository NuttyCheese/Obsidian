**UIDatePicker** — это встроенный контрол в [[UIKit]], который позволяет пользователю удобно выбирать дату, время или их комбинацию с помощью нативного [[iOS]]-интерфейса.

С iOS 14 (2020) он получил радикально новый внешний вид и поведение, а в 2026 году остаётся **единственным рекомендуемым** способом выбора даты/времени в UIKit-приложениях.

### 1. Основные стили отображения (preferredDatePickerStyle)

| Стиль                        | Внешний вид (iOS 14+)                              | Когда использовать в 2026 году                          | Где лучше всего смотрится |
|------------------------------|-----------------------------------------------------|----------------------------------------------------------|---------------------------|
| `.wheels`                    | Классические колёсики (как в iOS 13 и раньше)      | Когда нужно много точности (например, будильник)         | Формы, настройки времени  |
| `.compact`                   | Маленькое поле с календарем при тапе               | Экономия места, современный минималистичный дизайн       | Формы регистрации, фильтры |
| `.inline`                    | Полноэкранный календарь прямо на экране            | Когда выбор даты — основная задача экрана                | Календари, планировщики   |
| `.automatic` (по умолчанию)  | Система сама выбирает лучший стиль                 | Почти всегда — адаптируется под контекст                 | Универсальный выбор       |

### 2. Самый популярный паттерн 2026 года (рекомендуемый)

#### Вариант 1: Компактный пикер в форме (самый частый)

```swift
class BookingViewController: UIViewController {
    
    private let datePicker = UIDatePicker()
    private let dateLabel = UILabel()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        
        // Настройка пикера
        datePicker.datePickerMode = .date
        datePicker.preferredDatePickerStyle = .compact
        datePicker.locale = Locale(identifier: "ru_RU")
        datePicker.minimumDate = Date()  // нельзя выбрать прошлое
        datePicker.addAction(UIAction { [weak self] _ in
            self?.updateDateLabel()
        }, for: .valueChanged)
        
        datePicker.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(datePicker)
        
        // Метка с выбранной датой
        dateLabel.text = "Выберите дату"
        dateLabel.font = .preferredFont(forTextStyle: .body)
        dateLabel.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(dateLabel)
        
        NSLayoutConstraint.activate([
            datePicker.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            datePicker.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 40),
            
            dateLabel.topAnchor.constraint(equalTo: datePicker.bottomAnchor, constant: 20),
            dateLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor)
        ])
        
        updateDateLabel()
    }
    
    private func updateDateLabel() {
        let formatter = DateFormatter()
        formatter.dateStyle = .long
        formatter.timeStyle = .none
        formatter.locale = Locale(identifier: "ru_RU")
        
        dateLabel.text = "Выбрано: \(formatter.string(from: datePicker.date))"
    }
}
```

#### Вариант 2: Inline-календарь (полноэкранный выбор даты)

```swift
class CalendarViewController: UIViewController {
    
    private let datePicker = UIDatePicker()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        
        datePicker.datePickerMode = .date
        datePicker.preferredDatePickerStyle = .inline
        datePicker.locale = Locale(identifier: "ru_RU")
        
        // Ограничения: только будущие даты + 1 год вперёд
        datePicker.minimumDate = Date()
        datePicker.maximumDate = Calendar.current.date(byAdding: .year, value: 1, to: Date())
        
        datePicker.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(datePicker)
        
        NSLayoutConstraint.activate([
            datePicker.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20),
            datePicker.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            datePicker.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20),
            datePicker.bottomAnchor.constraint(lessThanOrEqualTo: view.safeAreaLayoutGuide.bottomAnchor, constant: -20)
        ])
        
        // Реакция на выбор
        datePicker.addAction(UIAction { [weak self] _ in
            self?.handleDateSelection(self?.datePicker.date ?? Date())
        }, for: .valueChanged)
    }
    
    private func handleDateSelection(_ date: Date) {
        let formatter = DateFormatter()
        formatter.dateStyle = .long
        print("Выбрана дата: \(formatter.string(from: date))")
        
        // Можно сразу закрыть экран или показать подтверждение
    }
}
```

### 3. Полный список ключевых свойств UIDatePicker

| Свойство / Метод                  | Тип / Возвращает                                | Что делает / Рекомендация 2026 |
|-----------------------------------|-------------------------------------------------|---------------------------------|
| `datePickerMode`                  | `.date`, `.time`, `.dateAndTime`, `.countDownTimer` | `.date` или `.dateAndTime` — самые частые |
| `preferredDatePickerStyle`        | `.wheels`, `.compact`, `.inline`, `.automatic`  | `.compact` — для форм, `.inline` — для календарей |
| `date`                            | `Date`                                          | Текущая выбранная дата/время |
| `minimumDate` / `maximumDate`     | `Date?`                                         | Ограничения диапазона (очень важно для UX) |
| `minuteInterval`                  | `Int` (1–30)                                    | Шаг минут (15 или 30 для будильников) |
| `locale`                          | `Locale?`                                       | `Locale(identifier: "ru_RU")` для русского формата |
| `calendar`                        | `Calendar`                                      | Можно задать григорианский, еврейский и т.д. |
| `timeZone`                        | `TimeZone?`                                     | Обычно не трогают (системный) |
| `countsDown`                      | `Bool` (только для `.countDownTimer`)           | Обратный отсчёт (таймер) |

### 4. Лучшие практики UIDatePicker в Swift 2026

- **Всегда** задавай `preferredDatePickerStyle` — `.automatic` оставляет на усмотрение системы, лучше явно  
- **Ограничивай диапазон** — `minimumDate` и `maximumDate` улучшают UX и предотвращают ошибки  
- **Локализация** — используй `locale` и `calendar` для корректного отображения в разных регионах  
- **UIAction вместо target-action** — современный стиль:

```swift
datePicker.addAction(UIAction { [weak self] _ in
    self?.handleDateChange(self?.datePicker.date ?? Date())
}, for: .valueChanged)
```

- **Не используй `.wheels` на iPhone** — он выглядит устаревшим, лучше `.compact`  
- **Для выбора диапазона дат** — используй два пикера или сторонние библиотеки (SwiftUI имеет встроенный RangeDatePicker)  
- **@MainActor** — все обновления UI — на главном акторе  
- **Swift 6 strict concurrency** — UIDatePicker полностью безопасен  
- **Документируйте** — пиши комментарий «UIDatePicker — компактный выбор даты с локализацией ru_RU»

**Короткий девиз 2026**:
> UIDatePicker — это **нативный и самый удобный** способ выбора даты/времени в UIKit.  
> В 2026 году используй **.compact** или **.inline**, **UIAction**, ограничения диапазона и правильную локализацию.  
> Это **единственный правильный** контрол для дат — забудь кастомные календари, если нет очень специфических требований.
