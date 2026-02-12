#third-party_library #architectural_approaches #Swift 
## 📘 Определение

**RxSwift** — это библиотека для **реактивного программирования на [[Swift]]**, основанная на **паттерне [[Observer]]**.  
Позволяет **работать с асинхронными потоками данных**, комбинировать события, обрабатывать ошибки и управлять состоянием приложения декларативно.  
Относится к **Reactive Programming / Networking / UI**.

---

## 🔹 Примеры кода

### 1. Простейший Observable

```swift
import RxSwift

let observable = Observable.of(1, 2, 3)
let disposable = observable.subscribe(onNext: { value in
    print("Получено значение:", value)
})
```

---

### 2. Подписка на события с onCompleted

```swift
let observable = Observable.of("A", "B", "C")

observable.subscribe(
    onNext: { print($0) },
    onCompleted: { print("Поток завершён") }
)
```

---

### 3. PublishSubject (аналог [[PassthroughSubject]])

```swift
let subject = PublishSubject<String>()

subject.onNext("До подписки") // не будет получено

subject.subscribe(onNext: { print("Подписчик получил:", $0) })

subject.onNext("Hello")
subject.onNext("World")
```

---

### 4. Observable с [[map]]

```swift
let numbers = Observable.of(1, 2, 3, 4)

numbers
    .map { $0 * 2 }
    .subscribe(onNext: { print($0) }) // 2, 4, 6, 8
```

---

### 5. [[combineLatest]]

```swift
let subject1 = BehaviorSubject(value: 1)
let subject2 = BehaviorSubject(value: 10)

Observable.combineLatest(subject1, subject2) { $0 + $1 }
    .subscribe(onNext: { print("Сумма:", $0) })

subject1.onNext(2)
subject2.onNext(20)
```

---

### 6. Использование с UI ([[UIButton]])

```swift
import UIKit
import RxCocoa

let button = UIButton()
let disposeBag = DisposeBag()

button.rx.tap
    .subscribe(onNext: { print("Кнопка нажата") })
    .disposed(by: disposeBag)
```
