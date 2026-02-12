#combine #reactive #Swift 
## 📘 Определение

**`debounce`** — оператор в [[Combine]] (и других реактивных фреймворках), который **откладывает выдачу события на заданное время** и передаёт его только если новые события не появились в течение этого интервала.  
Используется для **сглаживания потока данных**, например при вводе текста в поисковую строку или реагировании на быстрые последовательные события.

---

## 🔹 Примеры кода

### 1. Простое использование `debounce` с [[PassthroughSubject]]

```swift
import Combine
import Foundation

let subject = PassthroughSubject<String, Never>()
var cancellables = Set<AnyCancellable>()

subject
    .debounce(for: .seconds(1), scheduler: RunLoop.main)
    .sink { value in
        print("Debounced value:", value)
    }
    .store(in: &cancellables)

subject.send("H")
subject.send("He")
subject.send("Hel")
subject.send("Hell")
subject.send("Hello")
// через 1 секунду после последнего события напечатает "Hello"
```

---

### 2. Использование с `@Published` свойством

```swift
class ViewModel {
    @Published var searchText = ""
    var cancellables = Set<AnyCancellable>()

    init() {
        $searchText
            .debounce(for: .milliseconds(500), scheduler: RunLoop.main)
            .sink { text in
                print("Поиск по тексту:", text)
            }
            .store(in: &cancellables)
    }
}

let vm = ViewModel()
vm.searchText = "S"
vm.searchText = "Sw"
vm.searchText = "Swi"
vm.searchText = "Swif"
vm.searchText = "Swift"
```

---

### 3. `debounce` с асинхронной сетью

```swift
func fetchResults(for query: String) -> AnyPublisher<[String], Never> {
    Just(["Result for \(query)"]).eraseToAnyPublisher()
}

let searchText = PassthroughSubject<String, Never>()

searchText
    .debounce(for: .milliseconds(300), scheduler: RunLoop.main)
    .flatMap { query in
        fetchResults(for: query)
    }
    .sink { results in
        print("Результаты поиска:", results)
    }
    .store(in: &cancellables)

searchText.send("S")
searchText.send("Sw")
searchText.send("Swi")
searchText.send("Swift")
```

---

### 4. Использование с [[UITextField]] в [[UIKit]]

```swift
import UIKit
import Combine

let textField = UITextField()
var cancellables = Set<AnyCancellable>()

NotificationCenter.default.publisher(for: UITextField.textDidChangeNotification, object: textField)
    .compactMap { ($0.object as? UITextField)?.text }
    .debounce(for: .milliseconds(500), scheduler: RunLoop.main)
    .sink { text in
        print("Текст с дебаунсом:", text)
    }
    .store(in: &cancellables)
```

---

### 5. `debounce` с клавиатурным вводом в [[SwiftUI]]

```swift
import SwiftUI
import Combine

class SearchVM: ObservableObject {
    @Published var query = ""
    var cancellables = Set<AnyCancellable>()

    init() {
        $query
            .debounce(for: .milliseconds(300), scheduler: RunLoop.main)
            .sink { text in
                print("Поиск SwiftUI:", text)
            }
            .store(in: &cancellables)
    }
}

struct ContentView: View {
    @StateObject var vm = SearchVM()

    var body: some View {
        TextField("Поиск", text: $vm.query)
            .padding()
    }
}
```
