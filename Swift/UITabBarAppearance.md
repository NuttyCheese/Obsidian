Это современный способ (iOS 13+) кастомизировать [[UITabBar]].
`UITabBarAppearance` — это объект, описывающий **внешний вид** таб-бара:

- фон,
    
- разделители,
    
- тени,
    
- цвета и шрифты текста,
    
- иконки.
    

Он пришёл на замену старым свойствам (`barTintColor`, `shadowImage` и т.д.), чтобы внешний вид выглядел одинаково в **стандартном**, **компактном** и **прокручиваемом (scroll edge)** состояниях.

---

## 🔹 Основные состояния

У `UITabBarAppearance` есть три «слоя стиля» для разных ситуаций:

1. **`standardAppearance`** – обычный вид таб-бара.
    
2. **`compactAppearance`** – для компактного режима (например, в Split View на iPad).
    
3. **`scrollEdgeAppearance`** – когда таб-бар находится у края скролла (обычно в [[iOS]] 15+).
    

---

## 🔹 Настройка примера

```swift
let appearance = UITabBarAppearance()
appearance.configureWithOpaqueBackground() // или .configureWithDefaultBackground()

// 🔹 Настраиваем фон
appearance.backgroundColor = .white

// 🔹 Настраиваем иконки и текст для вкладок
appearance.stackedLayoutAppearance.normal.iconColor = .gray
appearance.stackedLayoutAppearance.normal.titleTextAttributes = [.foregroundColor: UIColor.gray]

appearance.stackedLayoutAppearance.selected.iconColor = .systemBlue
appearance.stackedLayoutAppearance.selected.titleTextAttributes = [.foregroundColor: UIColor.systemBlue]

// 🔹 Убираем верхнюю линию (тень)
appearance.shadowColor = .clear

// Применяем к TabBar
let tabBar = UITabBar()
tabBar.standardAppearance = appearance

if #available(iOS 15.0, *) {
    tabBar.scrollEdgeAppearance = appearance
}
```

---

## 🔹 Полезные методы

- `configureWithDefaultBackground()` → стандартный стиль.
    
- `configureWithOpaqueBackground()` → непрозрачный фон.
    
- `configureWithTransparentBackground()` → прозрачный фон.
    

---

## 🔹 Где это применимо

- Когда нужно **единое оформление** для всей навигации (например, светлый или тёмный стиль).
    
- Для **брендирования приложения** (свои цвета и шрифты).
    
- Для **iOS 15+**, где без `scrollEdgeAppearance` внешний вид может «прыгать» при скролле.
    

---
