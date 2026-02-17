**`Sequence`** — это **протокол**, который описывает тип, значения которого можно **перебирать по одному элементу за раз**.

- Все стандартные коллекции в [[Swift]] ([[Swift/Теория/Swift/Standart Library/Array]], [[Set]], [[Dictionary]], [[Range]]) соответствуют протоколу `Sequence`
    
- Для перебора используется метод `makeIterator()`, который возвращает **Iterator**
    
- Позволяет использовать [[for-in]] цикл для итерации
    

> Проще говоря: Sequence = «что-то, что можно перебирать элемент за элементом».

---

## 2. Основные термины

| Термин                              | Описание                                      |
| ----------------------------------- | --------------------------------------------- |
| **Sequence**                        | Протокол для типов, которые можно перебирать  |
| **Iterator**                        | Объект, который возвращает элементы по одному |
| **Element**                         | Тип элементов, которые содержит Sequence      |
| **for-in**                          | Цикл для перебора Sequence                    |
| **[[lazy]]**                        | Позволяет ленивую обработку элементов         |
| **[[map]], [[filter]], [[reduce]]** | Методы для работы с Sequence                  |

---

## 3. Основной синтаксис

```swift
let numbers: [Int] = [1, 2, 3]
for number in numbers {
    print(number)
}
```

- `Array` — Sequence
    
- Цикл `for-in` автоматически использует `makeIterator()`
    

---

## 4. Примеры от простого к сложному

### Пример 1. Простейший Array как Sequence

```swift
let numbers = [1, 2, 3, 4, 5]
for number in numbers {
    print(number)
}
```

- Все элементы перебираются по порядку
    

---

### Пример 2. Set как Sequence

```swift
let letters: Set = ["a", "b", "c"]
for letter in letters {
    print(letter)
}
```

- Порядок не гарантирован, но Set тоже Sequence
    

---

### Пример 3. Dictionary как Sequence

```swift
let dict = ["one": 1, "two": 2]
for (key, value) in dict {
    print("\(key): \(value)")
}
```

- Перебираются пары `(key, value)`
    

---

### Пример 4. Создание собственного Sequence

```swift
struct Countdown: Sequence {
    let start: Int
    
    func makeIterator() -> some IteratorProtocol {
        var value = start
        return AnyIterator {
            if value >= 0 {
                defer { value -= 1 }
                return value
            } else {
                return nil
            }
        }
    }
}

for i in Countdown(start: 3) {
    print(i)
}
```

- Пользовательский Sequence с IteratorProtocol
    

---

### Пример 5. Ленивый Sequence и методы map/filter

```swift
let numbers = [1, 2, 3, 4, 5]
let result = numbers.lazy
    .map { $0 * 2 }
    .filter { $0 > 5 }

for value in result {
    print(value)
}
```

- `lazy` позволяет обрабатывать элементы **по мере перебора**, экономя память
    

---

### Пример 6. Sequence с генератором (Infinite Sequence)

```swift
struct FibonacciSequence: Sequence {
    func makeIterator() -> AnyIterator<Int> {
        var a = 0, b = 1
        return AnyIterator {
            let next = a
            (a, b) = (b, a + b)
            return next
        }
    }
}

for num in FibonacciSequence() {
    if num > 50 { break }
    print(num)
}
```

- Генерация бесконечной последовательности
    
- Цикл `for-in` можно прерывать вручную
    

---

## 5. Особенности Sequence

1. **Одноразовая итерация** — Sequence можно перебрать несколько раз, но некоторые пользовательские Sequence могут быть одноразовыми
    
2. Все стандартные коллекции Swift соответствуют Sequence
    
3. Sequence предоставляет **базу для map, filter, reduce и других функций**
    
4. Можно создавать свои Sequence через **makeIterator и AnyIterator**
    
5. Ленивая обработка (`lazy`) позволяет экономить память на больших или бесконечных коллекциях
    

---

## 6. Итог

- **Sequence** = протокол для перебираемых коллекций
    
- Любой тип, соответствующий Sequence, можно использовать в **for-in цикле**
    
- Поддерживает **map, filter, reduce, lazy**
    
- Можно создавать **пользовательские и бесконечные последовательности**
    

---
