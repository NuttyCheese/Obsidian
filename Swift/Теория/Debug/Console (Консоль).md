![Image](https://i.sstatic.net/rWhcM.png)

![Image](https://a11y-guidelines.orange.com/en/mobile/images/iOSdev/wwdc21-xcodeLLDB-ViewDebuggingTips_1.png)

![Image](https://a11y-guidelines.orange.com/en/mobile/images/iOSdev/wwdc21-xcodeLLDB-print-p_1.png)

## Что такое Console

**Console** — это область отладки в [[Xcode]], где:

- выводятся логи приложения
    
- отображаются [[runtime]]-ошибки
    
- показываются системные сообщения [[iOS]]
    
- выполняются команды [[LLDB (Low-Level Debugger)|LLDB]]
    
- анализируется стек вызовов
    

Проще говоря:

> Консоль — это окно прямого взаимодействия с работающим приложением.

Она работает в связке с [[LLDB (Low-Level Debugger)|LLDB]] и системой логирования Apple.

---

# 📍 Где находится Console

В Xcode:

- `⌘ + Shift + Y` — показать/скрыть Debug Area
    
- Нижняя часть экрана — это Console
    
- Левая часть — стек потоков
    
- Правая — ввод команд LLDB
    

---

# 🧱 Что отображается в Console

### 1️⃣ Логи разработчика

```swift
print("User loaded")
debugPrint(model)
```

---

### 2️⃣ [[OSLog]] / Logger (современный способ)

```swift
import os

let logger = Logger(subsystem: "com.app", category: "network")
logger.info("Request started")
```

Преимущества:

- уровни логирования
    
- фильтрация
    
- не засоряет релиз
    

---

### 3️⃣ Системные сообщения

- [[Auto Layout]] warnings
    
- Main Thread Checker
    
- Runtime warnings
    
- Memory issues
    

---

### 4️⃣ LLDB команды

Во время паузы можно писать:

```
po self
bt
expr model.name = "Test"
```

---

# 🛠 Основные сценарии использования

---

## 🧠 1. Анализ логики

```swift
print("State:", state)
```

Проверка последовательности событий.

---

## 🧠 2. Диагностика сетевых запросов

```swift
logger.debug("Response: \(response)")
```

Можно отслеживать:

- статус код
    
- тело ответа
    
- ошибки декодирования
    

---

## 🧠 3. Поиск AutoLayout проблем

Консоль покажет:

```
Unable to simultaneously satisfy constraints.
```

Если добавить symbolic breakpoint:

```
UIViewAlertForUnsatisfiableConstraints
```

Можно ловить это сразу.

---

## 🧠 4. Проверка потоков

Main Thread Checker выводит предупреждение:

```
UI API called on background thread
```

---

# 🔬 Разница между print, debugPrint и dump

### print

- краткий вывод
    
- вызывает `description`
    

### debugPrint

- использует `debugDescription`
    
- более подробный вывод
    

### dump

- рекурсивно выводит структуру объекта
    

Пример:

```swift
dump(user)
```

---

# 📊 Фильтрация логов

В Console можно:

- искать по ключевым словам
    
- фильтровать по категории
    
- скрывать системные сообщения
    

Важно при больших проектах.

---

# ⚙️ Уровни логирования (Logger)

```swift
logger.trace()
logger.debug()
logger.info()
logger.notice()
logger.warning()
logger.error()
logger.critical()
```

Правильный подход:

- trace/debug → разработка
    
- info → важные события
    
- warning → потенциальные проблемы
    
- error → реальные ошибки
    

---

# 🚀 Продвинутые возможности

---

## 🔥 1. Logpoints (через breakpoint)

Можно добавить действие:

```
po model
```

И включить "Automatically continue".

Работает как условный логгер.

---

## 🔥 2. Сохранение логов

Можно:

- экспортировать лог с устройства
    
- использовать Console.app (macOS)
    
- анализировать краши через Device Logs
    

---

## 🔥 3. Работа с симулятором и устройством

В реальном устройстве:

- логов больше
    
- могут появляться системные предупреждения
    
- лучше ловятся race conditions
    

---

# ⚠️ Частые ошибки

### ❌ Использовать только print

В больших проектах нужно структурированное логирование.

---

### ❌ Оставлять чувствительные данные в логах

Пароли, токены — нельзя.

---

### ❌ Полагаться только на логи

Логи показывают прошлое,  
Debugger — текущее состояние.

---

# 📈 Как правильно использовать Console

1. Логируй ключевые события (не всё подряд)
    
2. Используй уровни логирования
    
3. Фильтруй вывод
    
4. Для сложных багов переходи к breakpoint
    
5. Не оставляй лишние print в production
    

---

# 🎯 Когда Console особенно полезна

- разработка сетевого слоя
    
- отладка бизнес-логики
    
- поиск [[race condition]]
    
- проверка жизненного цикла ViewController
    
- диагностика падений
    

---
