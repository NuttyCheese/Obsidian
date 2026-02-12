# 📘 Force Unwrapping в Swift

**Force Unwrapping** — это способ получить значение из Optional, когда мы **уверены**, что там **не nil**.

В [[Swift]] [[Optional]] — это **enum**, который может содержать либо значение `.some(value)`, либо `.none` ([[nil]]).

---

## 🔹 Синтаксис

```swift
var name: String? = "Alice"
print(name!) // "Alice"
```

- Символ `!` после Optional → **force unwrap**
    
- Если значение **nil** → runtime ошибка **fatal error: unexpectedly found nil**
    

---

## 🔹 Примеры использования

### 1. Безопасное значение

```swift
var age: Int? = 25
print(age!) // 25
```

- Работает, потому что `age != nil`.
    

### 2. Когда Optional равен nil

```swift
var city: String? = nil
print(city!) // Fatal error: unexpectedly found nil while unwrapping an Optional value
```

- Force unwrapping вызывает **crash**, если Optional nil.
    

### 3. Чаще всего используется с IBOutlet

```swift
@IBOutlet weak var label: UILabel!
label.text = "Hello World" // label! будет implicit unwrapped
```

- IBOutlets из storyboard/xib часто имеют тип **implicitly unwrapped Optional** (`UILabel!`)
    
- Мы уверены, что после загрузки view они **не будут nil**, поэтому force unwrapping безопасен в этом контексте.
    

---

## 🔹 Force unwrapping vs Optional Binding

```swift
let name: String? = "Alice"

// Force unwrap
print(name!) // "Alice"

// Optional binding
if let unwrappedName = name {
    print(unwrappedName)
}
```

- **Force unwrap** → риск crash
    
- **Optional binding** → безопасно, nil обрабатывается
    

---

## 🔹 Под капотом

Optional в Swift — это:

```swift
enum Optional<Wrapped> {
    case none
    case some(Wrapped)
}
```

Force unwrapping выполняет:

```swift
switch optionalValue {
case .some(let value):
    return value
case .none:
    fatalError("Unexpected nil")
}
```

- Поэтому при nil сразу падает приложение.
    
- Компилятор не проверяет на nil, **runtime проверка обязательна**.
    

---

## 🔹 Implicitly Unwrapped Optional (IUO)

```swift
var label: UILabel! // implicitly unwrapped
label.text = "Hi"   // эквивалентно label!.text
```

- IUO — это **Optional**, который **можно использовать как обычное значение**, но при nil вызовет crash.
    
- Удобно для IBOutlet и инициализации после конструктора.
    

---

## 🔹 Когда использовать force unwrapping

✅ Когда **вы точно знаете**, что значение существует:

- IBOutlet после загрузки view
    
- Значение сразу после инициализации или проверки
    
- Тестовые данные, где nil невозможно
    

❌ Не использовать для пользовательского ввода или внешних данных.

---

## 🔹 Альтернативы force unwrapping

1. **Optional binding ([[if let]] / [[guard let]])**
    

```swift
if let name = name {
    print(name)
} else {
    print("Nil value")
}
```

2. **Nil-coalescing оператор `??`**
    

```swift
let displayName = name ?? "Anonymous"
print(displayName)
```

3. **Optional chaining**
    

```swift
print(name?.uppercased() ?? "No name")
```

---

## 🔹 Итог

- `!` → force unwrap, мгновенно достаёт значение Optional
    
- Опасно, может вызвать crash при nil
    
- Используется **только если уверены в существовании значения**
    
- Safe alternatives: `if let`, `guard let`, `??`, optional chaining
    

---
