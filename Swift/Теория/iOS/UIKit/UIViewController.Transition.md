**UIViewController.Transition** — это не отдельный класс или протокол, а общий термин, который в документации и разговорах разработчиков обычно обозначает:

- **Способы** и **механизмы** перехода (transition) от одного [[UIViewController]] к другому  
- В основном речь идёт о **модальных переходах** (present / dismiss) и **навигационных переходах** (push / pop в [[UINavigationController]])

В UIKit существует несколько уровней управления переходами. Вот актуальная картина на 2026 год с приоритетами и рекомендациями.

### 1. Самые часто используемые способы перехода (2026)

| Тип перехода                              | Как вызвать                                   | Уровень кастомизации | Поддержка interactive | Рекомендуется в 2026 | Примечание |
|-------------------------------------------|-----------------------------------------------|-----------------------|------------------------|-----------------------|------------|
| **Стандартный модальный (present)**       | `present(vc, animated: true)`                 | Низкий                | Нет                    | ★★★☆☆                 | crossDissolve / coverVertical |
| **UINavigationController push / pop**     | `navigationController?.pushViewController`    | Низкий                | Нет                    | ★★★★☆                 | Стандартный стек навигации |
| **UISheetPresentationController**         | `vc.modalPresentationStyle = .pageSheet`      | Средний               | Да (swipe down)        | ★★★★★                 | iOS 15+ — современный bottom sheet |
| **UIViewControllerTransitioningDelegate** | `vc.transitioningDelegate = customDelegate`   | Высокий               | Да                     | ★★★★★                 | Полная кастомизация анимации |
| **UINavigationControllerDelegate**        | `navigationController?.delegate = custom`     | Высокий               | Да                     | ★★★★☆                 | Кастомный push/pop |
| **Custom presentation controller**        | `UIPresentationController` subclass           | Очень высокий         | Да                     | ★★★☆☆                 | Редко, только если нужен нестандартный layout |

### 2. Самые актуальные и рекомендуемые варианты в 2026 году

#### A. Самый популярный в новых приложениях — [[UISheetPresentationController]] (iOS 15+)

```swift
let sheetVC = DetailViewController()
sheetVC.modalPresentationStyle = .pageSheet

if let sheet = sheetVC.sheetPresentationController {
    sheet.detents = [.medium(), .large()]           // высота: половина экрана / полный экран
    sheet.prefersGrabberVisible = true              // полоска для свайпа
    sheet.prefersScrollingExpandsWhenScrolledToEdge = false
    sheet.largestUndimmedDetentIdentifier = .medium // затемнение только при полном экране
    sheet.prefersEdgeAttachedInCompactHeight = true // прикрепление к краю в компактном режиме
}

present(sheetVC, animated: true)
```

**Почему это №1 в 2026**  
- Нативный bottom sheet / half sheet  
- Поддержка swipe-to-dismiss  
- Адаптируется под iPhone / iPad / Stage Manager  
- Минимальный код

#### B. Полная кастомная анимация — [[UIViewControllerTransitioningDelegate]]

(подробный пример в предыдущем ответе)

Когда нужен нестандартный эффект:
- zoom / scale из ячейки таблицы  
- slide from bottom с кастомной кривой  
- interactive drag-to-dismiss  
- анимация с blur / mask

#### C. Кастомный push / pop в навигации — [[UINavigationControllerDelegate]]

```swift
class CustomNavigationDelegate: NSObject, UINavigationControllerDelegate {
    
    func navigationController(_ navigationController: UINavigationController,
                              animationControllerFor operation: UINavigationController.Operation,
                              from fromVC: UIViewController,
                              to toVC: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        if operation == .push {
            return PushAnimator()
        } else if operation == .pop {
            return PopAnimator()
        }
        return nil
    }
    
    func navigationController(_ navigationController: UINavigationController,
                              interactionControllerFor animationController: UIViewControllerAnimatedTransitioning) -> UIViewControllerInteractiveTransitioning? {
        // swipe from left edge to pop
        return interactivePopController
    }
}

navigationController?.delegate = customDelegate
```

### Рекомендации по выбору перехода в 2026 году

| Задача / желаемый эффект                          | Лучший выбор в 2026                              | Минимальная версия iOS | Уровень сложности |
|---------------------------------------------------|--------------------------------------------------|--------------------------|-------------------|
| Обычный модальный экран                           | `present(vc, animated: true)`                    | iOS 13+                  | ★☆☆☆☆             |
| Современный bottom sheet / card                   | `modalPresentationStyle = .pageSheet` + detents  | iOS 15+                  | ★★☆☆☆             |
| Bottom sheet с очень кастомным видом и анимацией  | `UIViewControllerTransitioningDelegate`          | iOS 13+                  | ★★★★☆             |
| Стандартный push / pop в навигации                | `navigationController?.pushViewController`       | iOS 13+                  | ★☆☆☆☆             |
| Красивый кастомный push/pop                       | `UINavigationControllerDelegate`                 | iOS 13+                  | ★★★★☆             |
| Полноэкранный переход с зумом из ячейки           | `UIViewControllerTransitioningDelegate`          | iOS 13+                  | ★★★★★             |
| Интерактивный swipe-to-dismiss                    | `UIViewControllerTransitioningDelegate` + interactive | iOS 13+               | ★★★★★             |

### Короткий итог 2026

> Переходы между `UIViewController` в UIKit управляются несколькими механизмами.  
> В 2026 году приоритет такой:  
> 1. **UISheetPresentationController** — для большинства модальных экранов (bottom sheet, half sheet)  
> 2. **UIViewControllerTransitioningDelegate** — когда нужна полная кастомная анимация и/или интерактивность  
> 3. **UINavigationControllerDelegate** — для кастомного push/pop в навигационном стеке  
> 4. Обычный `present` / `push` — когда стандартный вид устраивает  
