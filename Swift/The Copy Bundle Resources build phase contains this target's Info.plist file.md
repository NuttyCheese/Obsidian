[[Xcode]] предупреждает, что файл **Info.plist** вашего таргета случайно добавлен в фазу **Copy Bundle Resources**.

- `Info.plist` **не должен копироваться в бандл** приложения, так как Xcode обрабатывает его автоматически.
    
- Если оставить его в Copy Bundle Resources, это может привести к конфликтам, дублированию или предупреждениям при сборке.
    

---

### Примеры кода/сценариев возникновения

**Пример:**

- В Target → Build Phases → Copy Bundle Resources добавлен файл `Info.plist`.
    
- При сборке появляется предупреждение:
    

```
The Copy Bundle Resources build phase contains this target's Info.plist file
```

---

### Как исправить

#### 1️⃣ Удалить Info.plist из Copy Bundle Resources

1. Target → Build Phases → **Copy Bundle Resources**
    
2. Найти `Info.plist`
    
3. Нажать **минус (-)**, чтобы удалить
    

- Xcode продолжит использовать Info.plist автоматически для конфигурации таргета.
    

#### 2️⃣ Проверить правильность Target Membership

- Убедитесь, что Info.plist привязан к таргету через **Build Settings → Info.plist File**, а не через Copy Bundle Resources.
    

---

### Пример исправления

**Было:**

- Info.plist случайно добавлен в Copy Bundle Resources → предупреждение.
    

**Исправлено:**

- Info.plist удалён из Copy Bundle Resources
    
- Target → Build Settings → Info.plist File указан корректно:
    

```
$(SRCROOT)/ProjectName/Info.plist
```

- Предупреждение исчезает, сборка нормальная.
    

---
