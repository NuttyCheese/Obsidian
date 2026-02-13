## 1. Что такое `UINavigationBar`

**`UINavigationBar`** — это верхняя панель в [[iOS]]-приложении, которая обычно используется для:

- показа **заголовка текущего экрана**
    
- отображения **кнопок навигации** (например, "Назад", "Закрыть")
    
- размещения **дополнительных кнопок** (например, "Настройки", "Поиск")
    

Обычно `UINavigationBar` используется **вместе с** [[UINavigationController]], который управляет стеком экранов.

---

## 2. Основные элементы

На `UINavigationBar` могут находиться:

- **title** — заголовок экрана
    
- **leftBarButtonItems** — кнопки слева
    
- **rightBarButtonItems** — кнопки справа
    
- **back button** — кнопка "назад" (автоматически, если есть предыдущий экран)
    
- **prompt** — дополнительный текст над заголовком
    

---

## 3. Пример использования

### Пример 1. Заголовок

```swift
navigationItem.title = "Главная"
```

👉 Покажет заголовок в центре `UINavigationBar`.

---

### Пример 2. Кнопка "Настройки" справа

```swift
let settingsButton = UIBarButtonItem(
    image: UIImage(systemName: "gear"),
    style: .plain,
    target: self,
    action: #selector(openSettings)
)

navigationItem.rightBarButtonItem = settingsButton

@objc func openSettings() {
    print("Открыты настройки")
}
```

---

### Пример 3. Кнопка "Закрыть" слева

```swift
let closeButton = UIBarButtonItem(
    title: "Закрыть",
    style: .done,
    target: self,
    action: #selector(closeScreen)
)

navigationItem.leftBarButtonItem = closeButton

@objc func closeScreen() {
    dismiss(animated: true, completion: nil)
}
```

---

## 4. Кастомизация внешнего вида

`UINavigationBar` можно менять под дизайн приложения:

```swift
let appearance = UINavigationBarAppearance()
appearance.backgroundColor = .systemBlue
appearance.titleTextAttributes = [.foregroundColor: UIColor.white]
appearance.largeTitleTextAttributes = [.foregroundColor: UIColor.white]

// применяем
navigationController?.navigationBar.standardAppearance = appearance
navigationController?.navigationBar.scrollEdgeAppearance = appearance
```

👉 Теперь `UINavigationBar` будет синим, с белыми заголовками.

---

## 5. Особенности

- Обычно **не добавляют `UINavigationBar` вручную** — за это отвечает `UINavigationController`.
    
- Если нужен отдельный кастомный бар, можно использовать [[UINavigationBar]] как обычный [[Swift/Теория/UIKit/UIView]].
    
- Поддерживает **Large Titles** (большие заголовки при скролле списка).
    

```swift
navigationController?.navigationBar.prefersLargeTitles = true
navigationItem.largeTitleDisplayMode = .always
```

---

## 6. Итог

- `UINavigationBar` = верхняя панель для навигации и действий.
    
- Чаще всего управляется через `UINavigationController`.
    
- Поддерживает заголовки, кнопки и кастомизацию внешнего вида.
    
- Можно использовать `UINavigationBarAppearance` для единообразного стиля.
    

---
