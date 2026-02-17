Это предупреждение появляется, когда вы пытаетесь **показать (present) [[UIViewController]]**, который **ещё не добавлен в иерархию окон (window hierarchy)**.

- Обычно возникает при использовании `present(_:animated:completion:)`.
    
- `UIViewController` должен быть **видимым или уже добавленным в иерархию view** в момент вызова.
    
- Если его ещё нет — [[iOS]] не может корректно показать контроллер → предупреждение и потенциальный краш.
    

---

### Примеры кода/сценариев возникновения

**Пример 1: Вызов `present` в [[viewDidLoad]]**

```swift
class MyViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        let alert = UIAlertController(title: "Hello", message: "World", preferredStyle: .alert)
        present(alert, animated: true)  // ⚠️ Attempt to present … whose view is not in the window hierarchy
    }
}
```

- В `viewDidLoad` view ещё не добавлен в окно → предупреждение.
    

---

**Пример 2: Асинхронный вызов на уже закрытом контроллере**

```swift
func fetchData() {
    NetworkManager.shared.getData { [weak self] data in
        let vc = DetailViewController()
        self?.present(vc, animated: true)  // ⚠️ если self уже удалён, view не в иерархии
    }
}
```

- Если контроллер [[self]] уже не на экране, `present` не сработает → предупреждение.
    

---

### Как исправить

#### 1️⃣ Использовать [[viewDidAppear]] вместо `viewDidLoad`

```swift
override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    let alert = UIAlertController(title: "Hello", message: "World", preferredStyle: .alert)
    present(alert, animated: true) // ✅ view уже в иерархии
}
```

---

#### 2️⃣ Проверять `self.isViewLoaded` и `view.window != nil`

```swift
if self.isViewLoaded && self.view.window != nil {
    present(alert, animated: true) // ✅ только если view в иерархии
}
```

---

#### 3️⃣ Асинхронные вызовы через DispatchQueue.main.async

- Иногда view ещё не добавлен, но добавление в очередь позволяет корректно показать:
    

```swift
DispatchQueue.main.async {
    self.present(alert, animated: true)
}
```

---

#### 4️⃣ Использовать корректный presenting controller

- Убедитесь, что вы вызываете `present` от **видимого контроллера**, а не от закрытого или ещё не показанного.
    

```swift
UIApplication.shared.keyWindow?.rootViewController?.present(alert, animated: true)
```

- Только для особых случаев, лучше использовать текущий видимый контроллер.
    

---

### Резюме

- Предупреждение появляется, когда `present` вызывается на контроллере, **не добавленном в иерархию view**.
    
- Исправляется вызовом `present` в `viewDidAppear`, проверкой `view.window != nil` или использованием DispatchQueue.main.async.
    
- Помогает избежать крашей и неправильного отображения UI.
    

---
