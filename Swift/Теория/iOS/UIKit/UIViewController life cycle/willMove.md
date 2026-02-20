**`willMoveToSuperview`** и **`willMoveToWindow`** — это два метода жизненного цикла `UIView` в UIKit, которые вызываются **непосредственно перед** тем, как представление добавляется или удаляется из иерархии.

Они дают возможность **подготовиться** к изменению или **отменить** его (в случае `willMoveToSuperview`).

### Основные методы

| Метод                              | Когда вызывается                                                                 | Можно ли отменить изменение? | Самые частые сценарии использования в 2026 |
|------------------------------------|----------------------------------------------------------------------------------|------------------------------|---------------------------------------------|
| `willMoveToSuperview(newSuperview:)` | Перед тем, как вью добавят в новый superview или удалят из текущего (newSuperview может быть nil) | **Да** (если вызвать `removeFromSuperview()` внутри) | Отмена добавления, подготовка к удалению, очистка KVO/наблюдателей |
| `willMoveToWindow(newWindow:)`     | Перед тем, как вью добавят в новое окно или удалят из текущего (newWindow может быть nil) | **Нет** (отменить нельзя)    | Подготовка к появлению/исчезновению, остановка таймеров/анимаций, которые нельзя прерывать |

### Порядок вызовов при добавлении/удалении

#### Добавление в иерархию

1. `willMoveToSuperview(newSuperview)`  
2. добавление в superview  
3. `didMoveToSuperview()`  
4. `willMoveToWindow(newWindow)`  
5. добавление в окно  
6. `didMoveToWindow()`

#### Удаление из иерархии

1. `willMoveToSuperview(nil)`  
2. удаление из superview  
3. `didMoveToSuperview()`  
4. `willMoveToWindow(nil)`  
5. удаление из окна  
6. `didMoveToWindow()`

### Самые популярные и полезные паттерны в 2026 году

#### 1. Отмена добавления в определённый superview (самый частый кейс для willMoveToSuperview)

```swift
override func willMoveToSuperview(_ newSuperview: UIView?) {
    super.willMoveToSuperview(newSuperview)
    
    // Запрещаем добавлять эту вью в определённый контейнер
    if let forbidden = newSuperview, forbidden.tag == 999 {
        removeFromSuperview()  // ← отмена добавления
        print("Запрещено добавлять в superview с tag 999")
    }
}
```

#### 2. Очистка ресурсов перед удалением (очень важный сценарий)

```swift
override func willMoveToSuperview(_ newSuperview: UIView?) {
    super.willMoveToSuperview(newSuperview)
    
    if newSuperview == nil {
        // Вью сейчас удаляют → очищаем всё, что может вызвать retain cycle
        stopTimer()
        removeObservers()
        stopAVPlayer()
        cancelImageLoading()
    }
}
```

#### 3. Подготовка перед появлением в окне (willMoveToWindow)

```swift
override func willMoveToWindow(_ newWindow: UIWindow?) {
    super.willMoveToWindow(newWindow)
    
    if newWindow != nil {
        // Вью сейчас добавят в окно → готовимся к появлению
        prepareForAppearance()
    } else {
        // Вью сейчас удалят из окна → экстренная остановка
        pauseAnimations()
        pauseVideo()
    }
}
```

### Лучшие практики willMoveTo... в Swift 2026

- **Используйте** `willMoveToSuperview` для:
  - **отмены** нежелательного добавления
  - очистки перед удалением (KVO, NotificationCenter, таймеры)
- **Используйте** `willMoveToWindow` для:
  - подготовки к первому появлению на экране
  - экстренной остановки ресурсов, которые **нельзя** прерывать в `didMoveToWindow(nil)`
- **Всегда** вызывайте `super.willMoveTo...`
- **Не делайте** тяжёлые операции (сеть, диск) — метод может вызываться очень часто
- **Для SwiftUI** — аналогов нет (используйте `.onAppear` / `.onDisappear` или `UIViewRepresentable` lifecycle)
- **Документируйте** — пишите комментарий «willMoveToSuperview — отмена добавления в запрещённый superview и очистка перед удалением»

**Короткий итог 2026**:
> `willMoveToSuperview` и `willMoveToWindow` — это **предупреждение** о предстоящем изменении иерархии/окна.  
> В 2026 году:  
> - `willMoveToSuperview` — можно **отменить** добавление (`removeFromSuperview()`) и очистить перед удалением  
> - `willMoveToWindow` — **нельзя** отменить, но можно подготовиться к появлению/исчезновению  
> - вызывайте `super`  
> - это **очень мощные** методы для предотвращения утечек памяти и контроля добавления/удаления вью  

Удачи с чистым и безопасным управлением жизненным циклом представлений в твоём проекте! 🧹