**`UINavigationController`** — это контейнерный контроллер (container view controller), который управляет **стеком экранов** ([[UIViewController]]) и предоставляет пользователю удобную навигацию между ними.

🔑 Особенности:

- Работает по принципу **стека (LIFO)**: новый экран помещается (`push`) поверх предыдущего, а возврат (`pop`) снимает его со стека.
    
- Автоматически добавляет сверху **[[UINavigationBar]]** с заголовком и кнопкой "Назад".
    
- Часто используется для иерархической навигации (например: Главная → Детали → Настройки).
    

---

## 2. Как работает стек

- `pushViewController(_:animated:)` → открыть новый экран
    
- `popViewController(animated:)` → вернуться назад
    
- `popToRootViewController(animated:)` → вернуться на главный экран
    

---

## 3. Пример использования

### Инициализация в AppDelegate / SceneDelegate

```swift
let rootVC = HomeViewController()
let navigationController = UINavigationController(rootViewController: rootVC)
window?.rootViewController = navigationController
window?.makeKeyAndVisible()
```

👉 Теперь `HomeViewController` будет **первым экраном** в стеке.

---

### Переход на новый экран

```swift
let detailsVC = DetailsViewController()
navigationController?.pushViewController(detailsVC, animated: true)
```

---

### Вернуться назад

```swift
navigationController?.popViewController(animated: true)
```

---

### Вернуться на главный экран

```swift
navigationController?.popToRootViewController(animated: true)
```

---

## 4. Навигационная панель (`UINavigationBar`)

Каждый экран в `UINavigationController` имеет свойство `navigationItem`, где можно настроить:

- `title` — заголовок
    
- `leftBarButtonItem` / `rightBarButtonItem` — кнопки
    
- `backButtonTitle` — текст кнопки "Назад"
    

Пример:

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    navigationItem.title = "Детали"
    navigationItem.rightBarButtonItem = UIBarButtonItem(
        barButtonSystemItem: .add,
        target: self,
        action: #selector(addItem)
    )
}
```

---

## 5. Кастомизация `UINavigationController`

Через `UINavigationBarAppearance` можно настраивать цвет, фон, шрифт и стиль заголовков:

```swift
let appearance = UINavigationBarAppearance()
appearance.backgroundColor = .systemBlue
appearance.titleTextAttributes = [.foregroundColor: UIColor.white]

let navController = UINavigationController(rootViewController: rootVC)
navController.navigationBar.standardAppearance = appearance
navController.navigationBar.scrollEdgeAppearance = appearance
```

---

## 6. Важные моменты

- В iOS приложения обычно используют **один главный `UINavigationController`**, но при необходимости можно создавать вложенные.
    
- `UINavigationController` часто комбинируют с [[UITabBarController]] → каждая вкладка имеет свой стек экранов.
    
- Поддерживает **swipe-to-go-back** (жест "свайп назад").
    

---

## 7. Итог

- `UINavigationController` = контейнер для организации переходов по стеку.
    
- Управляет **последовательной навигацией** (вперёд/назад).
    
- Автоматически создаёт и управляет **`UINavigationBar`**.
    
- Позволяет легко кастомизировать стиль и кнопки.
    

---
