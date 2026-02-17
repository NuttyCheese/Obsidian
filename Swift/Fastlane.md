## 📘 Определение

**Fastlane** — инструмент автоматизации [[iOS]] и Android проектов.  
Позволяет **автоматизировать сборку, тестирование, бета- и App Store релизы, управление сертификатами и профилями**, а также запуск скриптов и генерацию скриншотов.  
Часто используется в сочетании с **[[Xcode]]**, **[[Git]]**, **[[CI]]/[[CD]]** системами.

---

## 🔹 Примеры кода

### 1. Установка Fastlane через Homebrew

```bash
brew install fastlane
```

---

### 2. Инициализация Fastlane в проекте

```bash
cd MyApp
fastlane init
```

---

### 3. Простейший `Fastfile` для сборки приложения

```ruby
default_platform(:ios)

platform :ios do
  desc "Сборка и архив приложения"
  lane :build do
    gym(scheme: "MyApp") # сборка Xcode проекта
  end
end
```

---

### 4. lane для бета-релиза через [[TestFlight]]

```ruby
lane :beta do
  increment_build_number
  build_app(scheme: "MyApp")
  upload_to_testflight(username: "user@example.com")
end
```

---

### 5. lane для публикации в App Store

```ruby
lane :release do
  capture_screenshots
  build_app(scheme: "MyApp")
  upload_to_app_store(username: "user@example.com")
  slack(message: "Новая версия загружена в App Store")
end
```

---

### 6. lane с автоматическим управлением сертификатами

```ruby
lane :certificates do
  match(type: "appstore") # синхронизация сертификатов и профилей
end
```
