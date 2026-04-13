#uikit #uidynamics #animation #physics #ios #swift #uicollectionview

---

## UIDynamicAnimator — Физическая анимация в UIKit

### Определение

**`UIDynamicAnimator`** — это класс в [[UIKit]], который служит "движком" для физической анимации. Он отвечает за просчет физических взаимодействий (гравитация, столкновения, притяжение и т.д.) и применение рассчитанных параметров к анимируемым объектам в соответствии с законами классической механики .

Простыми словами, вы задаете физические законы (например, "на эту кнопку действует гравитация вниз"), а `UIDynamicAnimator` сам просчитывает все перемещения, ускорения и столкновения в каждом кадре.

### Зачем это знать [[iOS]]-разработчику?

1.  **Реалистичные анимации:** Позволяет создавать анимации, которые ведут себя как в реальном мире (эффекты падения, отскока, прилипания, рывка) .
2.  **Снижение сложности:** Вместо ручного расчета сложных кривых ускорения (ease-in, ease-out) вы просто описываете физическое поведение объекта.
3.  **Интерактивность:** Легко связывать анимацию с жестами пользователя (например, перетаскивание объекта с эффектом резинки).
4.  **Работа с коллекциями:** `UIDynamicAnimator` может управлять анимацией ячеек в `UICollectionView`, создавая эффекты "притяжения" или "отталкивания" при скролле .
5.  **Игровая механика:** Идеально подходит для казуальных игр или механик, где важны коллизии и гравитация.

---

### Архитектура и основные компоненты

В основе UIKit Dynamics лежат три ключевых элемента :

1.  **`UIDynamicAnimator` (Аниматор):** Контекст выполнения, который содержит все физические правила (поведения) для одной или нескольких областей экрана. Это "движок", который просчитывает физику.
2.  **[[UIDynamicBehavior]] (Поведение):** Правило или сила, которую вы хотите применить. Сами по себе поведения ничего не анимируют, они лишь описывают закон. Вы добавляете их в аниматор. Существует множество готовых подклассов (гравитация, коллизии и т.д.) .
3.  **[[UIDynamicItem]] (Элемент):** Объект, к которому применяется физика. По умолчанию любой `UIView` (и его наследники) соответствует протоколу `UIDynamicItem`. Вы добавляете элементы в конкретные поведения .

---

### Ключевые методы и свойства

| Метод / Свойство | Описание |
|---|---|
| **`init(referenceView: UIView)`** | Инициализирует аниматор с указанной **reference view**. Все координаты элементов задаются относительно этой вью . |
| **`addBehavior(_:)`** | Добавляет физическое поведение в аниматор . |
| **`removeBehavior(_:)`** / **`removeAllBehaviors()`** | Удаляет поведения . |
| **`updateItemUsingCurrentState(_:)`** | Сообщает аниматору, что вы вручную изменили состояние элемента (например, `center` или `transform`), и просит пересчитать физику на основе нового состояния . |
| **`isRunning`** | Указывает, находится ли аниматор в активном состоянии (просчитывает ли физику) . |

---

### Жизненный цикл и Delegate

Аниматор имеет четкие состояния, которые можно отслеживать через делегат:

```swift
class ViewController: UIViewController, UIDynamicAnimatorDelegate {
    var animator: UIDynamicAnimator!

    override func viewDidLoad() {
        super.viewDidLoad()
        animator = UIDynamicAnimator(referenceView: view)
        animator.delegate = self
    }

    // Вызывается, когда все движения остановились
    func dynamicAnimatorDidPause(_ animator: UIDynamicAnimator) {
        print("Физика успокоилась, все объекты остановились.")
    }

    // Вызывается, когда аниматор возобновляет работу (например, после приложения силы)
    func dynamicAnimatorWillResume(_ animator: UIDynamicAnimator) {
        print("Аниматор возобновил работу.")
    }
}
```
> **Важно:** Когда все добавленные в поведения элементы достигают состояния покоя, аниматор автоматически приостанавливается. При приложении новой силы или изменении состояния он возобновляет работу .

---

## Основные типы поведений (UIDynamicBehavior)

### 1. [[UIGravityBehavior]] (Гравитация)
Применяет силу гравитации к элементам.

```swift
let gravity = UIGravityBehavior(items: [myView])
// Настройка вектора и силы
gravity.gravityDirection = CGVector(dx: 0.0, dy: 1.0) // Вниз
gravity.magnitude = 1.0 // Сила притяжения

animator.addBehavior(gravity)
```

### 2. [[UICollisionBehavior]] (Столкновения)
Добавляет границы, об которые элементы могут ударяться.

```swift
let collision = UICollisionBehavior(items: [myView])
// Включить границы reference view как границы
collision.translatesReferenceBoundsIntoBoundary = true
// Добавить свою линию
collision.addBoundary(withIdentifier: "line" as NSCopying, 
                      from: CGPoint(x: 0, y: 300), 
                      to: CGPoint(x: 400, y: 300))
animator.addBehavior(collision)
```

### 3. [[UIAttachmentBehavior]] (Присоединение)
Связывает элемент с точкой или другим элементом (эффект "резинки").

```swift
// Привязка к точке на экране
let attachment = UIAttachmentBehavior(item: myView, attachedToAnchor: tapPoint)
attachment.length = 50.0 // Длина "паутинки"
attachment.damping = 0.5  // Демпфирование (чем выше, тем быстрее остановка)
attachment.frequency = 1.0 // Частота колебаний
animator.addBehavior(attachment)
```

### 4. [[UIPushBehavior]] (Толчок)
Прикладывает силу к элементу (мгновенную или постоянную).

```swift
// Мгновенный толчок
let push = UIPushBehavior(items: [myView], mode: .instantaneous)
push.angle = CGFloat.pi / 4 // Угол 45 градусов
push.magnitude = 0.5 // Сила толчка
animator.addBehavior(push)
```

### 5. [[UISnapBehavior]] (Притяжение)
"Телепортирует" элемент в указанную точку с эффектом пружины.

```swift
let snap = UISnapBehavior(item: myView, snapTo: targetPoint)
snap.damping = 0.3 // Уровень затухания (0.0 - бесконечная тряска)
animator.addBehavior(snap)
```

### 6. [[UIDynamicItemBehavior]] (Индивидуальные свойства)
Позволяет настроить физические свойства самого элемента (трение, эластичность, разрешить вращение).

```swift
let itemBehavior = UIDynamicItemBehavior(items: [myView])
itemBehavior.elasticity = 0.8 // Упругость при ударе (отскок)
itemBehavior.friction = 0.2   // Трение
itemBehavior.allowsRotation = true // Разрешить вращение при ударе
animator.addBehavior(itemBehavior)
```

---

## UIDynamicAnimator и [[UICollectionView]]

Начиная с iOS 7, аниматоры можно использовать для создания невероятно плавных и живых анимаций в [[UICollectionViewFlowLayout]] .

```swift
class CustomDynamicLayout: UICollectionViewFlowLayout {
    private var dynamicAnimator: UIDynamicAnimator?
    private var visibleIndexPathsSet = Set<IndexPath>()
    private var latestDelta: CGFloat = 0

    override func prepare() {
        super.prepare()
        guard let collectionView = collectionView, dynamicAnimator == nil else { return }

        // 1. Инициализируем аниматор с reference view равным collectionView
        dynamicAnimator = UIDynamicAnimator(collectionViewLayout: self)
        
        // 2. Создаем поведения для всех видимых ячеек
        let visibleRect = CGRect(origin: collectionView.bounds.origin, size: collectionView.frame.size)
        guard let layoutAttributes = super.layoutAttributesForElements(in: visibleRect) else { return }

        for attributes in layoutAttributes {
            let behavior = UIAttachmentBehavior(item: attributes, attachedToAnchor: attributes.center)
            behavior.length = 0
            behavior.damping = 0.8
            behavior.frequency = 1.0
            dynamicAnimator?.addBehavior(behavior)
            visibleIndexPathsSet.insert(attributes.indexPath)
        }
    }
    
    // Переопределяем метод для обработки скролла...
}
```

---

## Решение проблемы: `updateItemUsingCurrentState`

Классическая проблема: если вы вручную измените `transform` (скейл, поворот) у `UIView`, который находится под управлением `UIDynamicAnimator`, аниматор "сбросит" это изменение в следующем кадре, так как он не знает о вашем вмешательстве .

Чтобы аниматор учел ваши изменения, используйте метод **`updateItemUsingCurrentState`**:

```swift
// Например, увеличиваем ячейку, которая участвует в коллизиях
myView.transform = CGAffineTransform(scaleX: 1.5, y: 1.5)
// Сообщаем аниматору: "Обнови состояние этого элемента, я его изменил"
animator.updateItemUsingCurrentState(myView)
```

Это заставит аниматор перечитать текущий `center` и `transform` элемента и продолжить расчет физики уже с новыми параметрами .

---

### Лучшие практики

1.  **Один аниматор на reference view:** Обычно достаточно одного аниматора на вью, в которой происходит анимация .
2.  **Удаляйте поведения при уходе с экрана:** Если вью, к которому применена физика, удаляется из иерархии, обязательно удалите его из всех поведений и удалите поведения из аниматора, чтобы избежать утечек .
3.  **Используйте `action` для обратной связи:** У любого `UIDynamicBehavior` есть свойство `action`, которое вызывается в каждом кадре анимации. Это отличное место для обновления UI или триггера звуков .

```swift
gravity.action = {
    if myView.center.y > 500 {
        print("View упал ниже отметки")
    }
}
```

### Итог

**`UIDynamicAnimator`** — мощный инструмент для создания физически правдоподобных анимаций без сложных математических расчетов. Понимание его работы, типов поведений и особенностей (например, `updateItemUsingCurrentState`) позволяет добавлять в приложения высокую интерактивность и "живые" эффекты, которые сложно воспроизвести стандартными `UIView.animate` .