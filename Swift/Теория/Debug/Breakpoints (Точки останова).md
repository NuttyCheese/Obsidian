## 🛑 Breakpoints (Точки останова в [[Xcode]])

![Image](https://belief-driven-design.com/images/2023/2023-08-22-xcode-breakpoints-101-navigator.png)

![Image](https://www.delasign.com/CDN/images/Open-Navigator.webp)

![Image](https://i.sstatic.net/acGZ7.png)

![Image](https://i.sstatic.net/aAcaF.png)

## Что такое Breakpoint

**Breakpoint** — это точка в коде, где выполнение программы **временно останавливается**, чтобы ты мог:

- проверить значения переменных
    
- проанализировать стек вызовов
    
- выполнить код вручную
    
- изменить состояние приложения
    
- найти причину бага
    

Работают они через встроенный отладчик ([[LLDB (Low-Level Debugger)|LLDB]]).

Проще говоря:

> Breakpoint — это управляемая пауза в живом приложении.

---

# 📍 Основные виды Breakpoints

## 1️⃣ Line Breakpoint (строчный)

Самый распространённый.  
Ставится кликом в gutter слева от строки.

Останавливает выполнение **перед выполнением строки**.

Используется:

- для анализа логики
    
- для проверки параметров метода
    
- при отладке UI
    

---

## 2️⃣ Conditional Breakpoint (условный)

Останавливается **только если условие истинно**.

Пример:

```swift
index == 10
user.id == 123
isAuthorized == false
```

Очень полезно при:

- больших циклах
    
- [[UITableView]] / [[UICollectionView|CollectionView]]
    
- сложной бизнес-логике
    

---

## 3️⃣ Symbolic Breakpoint (символьный)

Ставится на имя функции или символа.

Примеры полезных:

- `UIViewAlertForUnsatisfiableConstraints`
    
- `objc_exception_throw`
    
- `Swift runtime failure`
    

Используется для:

- ловли [[Auto Layout]] ошибок
    
- поиска [[Runtime]] exception
    
- диагностики [[force unwrap]] crash
    

---

## 4️⃣ Exception Breakpoint

Останавливает выполнение при любом [[throw]] / crash.

Есть два типа:

- [[Objective-C]] Exception
    
- [[Swift]] Error
    

Полезен когда:

- приложение падает без понятной причины
    
- stacktrace не очевиден
    

---

## 5️⃣ Runtime Issue Breakpoints

Можно ловить:

- Thread sanitizer
    
- Main Thread Checker
    
- Memory issues
    

---

# ⚙️ Как настроить Breakpoint

ПКМ по breakpoint → Edit Breakpoint.

Можно настроить:

---

## 🧠 Condition (условие)

```
count > 50
view == nil
```

---

## 🧠 Ignore count

Игнорировать первые N срабатываний.

Полезно в циклах.

---

## 🧠 Action (действия)

Можно:

- выполнить LLDB команду (`po model`)
    
- вывести лог
    
- воспроизвести звук
    
- автоматически продолжить выполнение
    

💡 Это позволяет использовать breakpoint как логгер без print.

---

# 🎯 Практические сценарии в [[iOS]]

---

## 🧩 1. Отладка [[UITableView]]

Проблема: ячейки отображаются некорректно.

Ставим breakpoint в:

```swift
cellForRowAt
```

Проверяем:

```
po indexPath
po model[indexPath.row]
```

---

## 🧩 2. Проверка передачи данных между экранами

В [[viewDidLoad]] второго VC:

```
po model
po previousVC
```

Понимаем:

- данные пришли?
    
- не [[nil]] ли они?
    

---

## 🧩 3. Поиск [[retain cycle]]

Ставим breakpoint в:

```swift
deinit
```

Если не вызывается — объект удерживается.

---

## 🧩 4. Отладка [[async]] / completion

Ставим breakpoint внутри [[closure]]:

```swift
networkService.fetch { result in
```

Проверяем:

```
po result
```

---

## 🧩 5. AutoLayout ошибки

Создаём Symbolic Breakpoint:

```
UIViewAlertForUnsatisfiableConstraints
```

Приложение остановится в момент конфликта.

---

# 🔬 Глубокое понимание (что происходит внутри)

Когда выполнение доходит до breakpoint:

1. Процесс ставится на паузу
    
2. Поток замораживается
    
3. LLDB получает доступ к памяти
    
4. Ты можешь:
    
    - читать значения
        
    - менять их
        
    - выполнять код
        

Важно:  
Breakpoint останавливает **конкретный поток**, а не всё приложение.

---

# 🚀 Продвинутые приёмы

---

## 🔥 1. Logpoint (без остановки)

Добавь действие:

```
po model
```

И включи "Automatically continue after evaluating actions".

Получится мощный логгер.

---

## 🔥 2. Breakpoint на конкретный поток

Можно ограничить:

- [[main|main thread]]
    
- background queue
    

---

## 🔥 3. Breakpoint на метод

ПКМ по имени метода → Add Breakpoint.

Работает для:

- [[UIViewController]] [[Жизненный цикл ViewController-a|lifecycle]]
    
- [[delegate]] методов
    
- сетевых ответов
    

---

## 🔥 4. Temporary Breakpoint

Можно отключить (disable), не удаляя.

Очень удобно при поэтапной отладке.

---

# 🧠 Когда использовать Breakpoints вместо print

|print|breakpoint|
|---|---|
|показывает прошлое|показывает текущее состояние|
|засоряет код|не меняет код|
|нельзя изменить данные|можно изменить состояние|
|неудобен в async|работает везде|

---

# ⚠️ Частые ошибки

### ❌ Оставлять breakpoints в коммите

Перед push — отключать или удалять.

### ❌ Ставить слишком много

Делай точечно, иначе теряется логика анализа.

### ❌ Отлаживать в Release

В Release оптимизации мешают корректной работе.

---

# 📈 Правильный подход к отладке

1. Понять симптом
    
2. Определить предполагаемую точку проблемы
    
3. Поставить breakpoint
    
4. Проверить входные параметры
    
5. Проверить состояние объекта
    
6. Проверить стек вызовов
    
7. Только потом менять код
    

Так работали и 10 лет назад, и сейчас — меняются фреймворки, но дисциплина отладки остаётся прежней.

---
