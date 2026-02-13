## 1. Что такое `UITextFieldDelegate`

**`UITextFieldDelegate`** — это **протокол**, который предоставляет методы для реагирования на события **[[UITextField]]**:

- Начало редактирования
    
- Окончание редактирования
    
- Нажатие Return
    
- Ограничение символов и ввода
    
- Очистка текста
    

Используется через **свойство `delegate` текстового поля**.

> Проще говоря: `UITextFieldDelegate` = «обработчик событий UITextField».

---

## 2. Основные термины

|Термин|Описание|
|---|---|
|**delegate**|Свойство UITextField, куда присваивается объект, реализующий протокол|
|**textFieldShouldBeginEditing**|Вызывается перед началом редактирования, можно запретить его|
|**textFieldDidBeginEditing**|Вызывается после начала редактирования|
|**textFieldShouldEndEditing**|Вызывается перед завершением редактирования, можно запретить|
|**textFieldDidEndEditing**|Вызывается после завершения редактирования|
|**textField(_:shouldChangeCharactersIn:replacementString:)**|Позволяет контролировать ввод каждого символа|
|**textFieldShouldReturn**|Вызывается при нажатии клавиши Return|

---

## 3. Основной синтаксис

### Назначение делегата

```swift
let textField = UITextField()
textField.delegate = self // self должен соответствовать UITextFieldDelegate
```

### Реализация протокола

```swift
class ViewController: UIViewController, UITextFieldDelegate {
    func textFieldShouldReturn(_ textField: UITextField) -> Bool {
        textField.resignFirstResponder() // скрываем клавиатуру
        return true
    }
}
```

---

## 4. Примеры от простого к сложному

### Пример 1. Скрытие клавиатуры при Return

```swift
func textFieldShouldReturn(_ textField: UITextField) -> Bool {
    textField.resignFirstResponder()
    return true
}
```

- Простая реализация для скрытия клавиатуры
    

---

### Пример 2. Запрет редактирования

```swift
func textFieldShouldBeginEditing(_ textField: UITextField) -> Bool {
    return false // запрещаем редактирование
}
```

- Поле становится только для чтения
    

---

### Пример 3. Ограничение длины текста

```swift
func textField(_ textField: UITextField, shouldChangeCharactersIn range: NSRange, replacementString string: String) -> Bool {
    let maxLength = 10
    let currentString = (textField.text ?? "") as NSString
    let newString = currentString.replacingCharacters(in: range, with: string)
    return newString.count <= maxLength
}
```

- Пользователь не сможет ввести больше 10 символов
    

---

### Пример 4. Разрешение только чисел

```swift
func textField(_ textField: UITextField, shouldChangeCharactersIn range: NSRange, replacementString string: String) -> Bool {
    let allowedCharacters = CharacterSet.decimalDigits
    let characterSet = CharacterSet(charactersIn: string)
    return allowedCharacters.isSuperset(of: characterSet)
}
```

- Ограничиваем ввод только цифрами
    

---

### Пример 5. Динамическая реакция на текст

```swift
func textFieldDidChangeSelection(_ textField: UITextField) {
    print("Текущий текст: \(textField.text ?? "")")
}
```

- Срабатывает при каждом изменении выделения или текста
    

---

## 5. Особенности UITextFieldDelegate

1. **Делегат должен быть слабой ссылкой (`weak`)**, чтобы избежать retain cycle
    
2. Методы протокола **не обязательны**, их можно реализовать выборочно
    
3. Делегат позволяет **контролировать ввод, ограничения и реакции на действия пользователя**
    
4. Часто используется для:
    
    - Скрытия клавиатуры
        
    - Валидации ввода
        
    - Ограничения символов
        
    - Автоформатирования текста
        

---

## 6. Итог

- **UITextFieldDelegate** = протокол для обработки событий UITextField
    
- Позволяет:
    
    - Управлять началом и окончанием редактирования
        
    - Реагировать на Return
        
    - Ограничивать и фильтровать ввод
        
- Работает через свойство `delegate` текстового поля
    

---