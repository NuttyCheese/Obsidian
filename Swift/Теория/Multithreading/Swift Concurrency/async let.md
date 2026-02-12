## 1. Что такое `async let`

`async let` позволяет **создавать параллельные асинхронные задачи** внутри `async` контекста.

- Главное отличие от обычного [[await]]:
    
    - `await` выполняет функцию **последовательно**.
        
    - `async let` позволяет запустить несколько функций **параллельно**, а потом дождаться их результатов.
        
- Это удобно, когда задачи **не зависят друг от друга**, и нужно ускорить выполнение.
    

---

## 2. Синтаксис

```swift
async let имя = асинхронная_функция()
```

- `async let` создаёт параллельную задачу и сразу продолжает выполнение текущего кода.
    
- Чтобы получить результат, нужно использовать `await`:
    

```swift
let result = await имя
```

---

## 3. Примеры

### Пример 1. Две задачи параллельно

```swift
func fetchUser() async -> String {
    try? await Task.sleep(nanoseconds: 500_000_000)
    return "Пользователь Иван"
}

func fetchOrders() async -> String {
    try? await Task.sleep(nanoseconds: 500_000_000)
    return "Заказы: [#1, #2, #3]"
}

Task {
    async let user = fetchUser()
    async let orders = fetchOrders()
    
    // ждём оба результата
    let userResult = await user
    let ordersResult = await orders
    
    print(userResult)
    print(ordersResult)
}
```

✅ Обе функции выполняются **параллельно**, экономя время.

---

### Пример 2. С [[throws]]

```swift
enum NetworkError: Error {
    case failed
}

func fetchData1() async throws -> String {
    "Данные 1"
}

func fetchData2() async throws -> String {
    "Данные 2"
}

Task {
    async let first = try fetchData1()
    async let second = try fetchData2()
    
    do {
        let results = try await [first, second]
        print(results)
    } catch {
        print("Ошибка:", error)
    }
}
```

✅ Позволяет обрабатывать ошибки каждой параллельной задачи.

---

### Пример 3. Параллельная загрузка картинок

```swift
func loadImage(from url: String) async -> UIImage? {
    guard let url = URL(string: url),
          let data = try? Data(contentsOf: url),
          let image = UIImage(data: data)
    else { return nil }
    return image
}

Task {
    async let img1 = loadImage(from: "https://picsum.photos/200")
    async let img2 = loadImage(from: "https://picsum.photos/201")
    async let img3 = loadImage(from: "https://picsum.photos/202")
    
    let images = await [img1, img2, img3]
    print("Загружено \(images.compactMap { $0 }.count) изображений")
}
```

✅ Все загрузки идут одновременно.

---

### Пример 4. Смешивание с `await` для последовательности

```swift
func fetchProfile() async -> String { "Профиль" }
func fetchSettings() async -> String { "Настройки" }

Task {
    async let profile = fetchProfile()
    let settings = await fetchSettings() // ждём только settings
    
    let profileResult = await profile // ждём profile
    print(profileResult, settings)
}
```

👉 Можно комбинировать параллельные и последовательные вызовы.

---

### Пример 5. Использование в [[actor]]

```swift
actor DataLoader {
    func loadUser() async -> String { "Пользователь" }
    func loadOrders() async -> String { "Заказы" }
}

let loader = DataLoader()

Task {
    async let user = loader.loadUser()
    async let orders = loader.loadOrders()
    
    print(await user)
    print(await orders)
}
```

✅ `async let` отлично работает даже с actor, и все изоляции сохраняются.

---

## 4. Особенности `async let`

1. **Область видимости**
    
    - Переменные, созданные через `async let`, живут до конца текущего блока.
        
2. **Обязательный `await`**
    
    - Если не дождаться результата (`await`), компилятор выдаст предупреждение.
        
3. **Ошибки**
    
    - Если одна из задач бросает `throw`, `await` вернёт ошибку.
        
4. **Эффективность**
    
    - `async let` запускает задачи **параллельно** на уровне [[Swift]] Concurrency, не создавая отдельные `Task`, экономя накладные расходы.
        

---

## 5. Итог

- `async let` = способ **параллельного выполнения асинхронных функций**.
    
- Отличие от обычного `await`: параллелизм и экономия времени.
    
- Можно комбинировать с [[throws]], [[Task]], [[actor]].
    

---
