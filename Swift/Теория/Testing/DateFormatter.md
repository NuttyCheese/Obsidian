**`DateFormatter`** — это класс ([[Reference Type]]), который позволяет:

- Преобразовывать [[Date]] в строку для отображения пользователю.
    
- Преобразовывать строку в `Date` (парсинг).
    
- Устанавливать **формат даты**, **время**, **локаль** и **часовой пояс**.
    

> Проще: `DateFormatter` = мост между `Date` и читаемым временем.

---

## 2. Создание и базовое использование

```swift
let formatter = DateFormatter()
formatter.dateStyle = .medium   // формат даты
formatter.timeStyle = .short    // формат времени

let now = Date()
let dateString = formatter.string(from: now)
print(dateString)  // например, "27 Aug 2025, 12:34"
```

### Варианты `dateStyle` и `timeStyle`

|Стиль|Пример (27 августа 2025)|
|---|---|
|`.short`|27.08.25|
|`.medium`|27 Aug 2025|
|`.long`|27 August 2025|
|`.full`|Tuesday, 27 August 2025|

---

## 3. Форматирование через шаблон

Можно задать **свой формат** через `dateFormat`:

```swift
let formatter = DateFormatter()
formatter.dateFormat = "dd.MM.yyyy HH:mm:ss"
let now = Date()
print(formatter.string(from: now)) // 27.08.2025 12:34:56
```

- dd — день
    
- MM — месяц
    
- yyyy — год
    
- HH — часы (24)
    
- mm — минуты
    
- ss — секунды
    

---

## 4. Локализация и часовой пояс

```swift
formatter.locale = Locale(identifier: "ru_RU") // русский язык
formatter.timeZone = TimeZone(abbreviation: "GMT+3") // часовой пояс
```

Пример:

```swift
formatter.dateStyle = .full
formatter.timeStyle = .long
print(formatter.string(from: Date()))
// вторник, 27 августа 2025 г. 15:34:56 GMT+3
```

---

## 5. Парсинг строки в Date

```swift
let formatter = DateFormatter()
formatter.dateFormat = "dd/MM/yyyy HH:mm"
let dateString = "27/08/2025 12:34"

if let date = formatter.date(from: dateString) {
    print(date) // 2025-08-27 12:34:00 +0000
}
```

> Если строка не соответствует формату, `date(from:)` вернёт `nil`.

---

## 6. Особенности

- `DateFormatter` **дорогой в создании**, поэтому лучше хранить экземпляр и переиспользовать.
    
- Работает с `Date` и [[String]].
    
- Поддерживает локализацию и часовые пояса.
    
- Можно использовать предопределённые стили (`short`, `medium`, `long`, `full`) или собственный формат (`dateFormat`).
    

---

## 7. Итог

- **DateFormatter** нужен для отображения и парсинга дат.
    
- Используется вместе с `Date` и часто с [[Calendar]]/[[DateComponents]].
    
- Можно настроить **формат**, **локаль**, **часовой пояс**.
    
- Позволяет преобразовывать дату в строку для UI и обратно.
    

---
