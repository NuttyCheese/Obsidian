[[Xcode]] предупреждает, что **метод экземпляра вашего класса почти совпадает с опциональным методом протокола**, но **имя, сигнатура или параметры немного отличаются**.

- Опциональные методы встречаются в **[[Objective-C]] протоколах**, которые импортированы в [[Swift]].
    
- Swift пытается сопоставить ваш метод с требованием протокола, но обнаруживает **несоответствие** → выдаёт предупреждение.
    
- Обычно проблема возникает из-за:
    
    1. Различий в именовании (`camelCase` vs `snake_case`).
        
    2. Различий в типах параметров или возвращаемого значения.
        
    3. Различий в [[@objc]] аннотациях.
        

---

### Примеры кода/сценариев возникновения

**Пример 1: Несовпадение имени метода с [[Optional]] @objc протоколом**

```objc
// Objective-C
@protocol MyDelegate
@optional
- (void)didFinishTask:(NSInteger)taskId;
@end
```

```swift
class MyClass: NSObject, MyDelegate {
    func didFinishTask(taskID: Int) { 
        print("Finished task") // ⚠️ Instance method 'didFinishTask(taskID:)' nearly matches optional requirement 'didFinishTask:' of protocol 'MyDelegate'
    }
}
```

- Различие в параметре `taskId` vs `taskID` → предупреждение.
    

---

**Пример 2: Различие в сигнатуре метода**

```objc
@protocol MyDelegate
@optional
- (void)processData:(NSString *)data;
@end
```

```swift
class Processor: NSObject, MyDelegate {
    func processData(data: String?) { 
        print(data ?? "") // ⚠️ почти совпадает, но тип Optional отличается
    }
}
```

- Сигнатура должна точно совпадать с Objective-C протоколом → предупреждение.
    

---

### Как исправить

#### 1️⃣ Использовать точное имя и сигнатуру

```swift
class MyClass: NSObject, MyDelegate {
    func didFinishTask(_ taskId: Int) {  // ✅ соответствует протоколу
        print("Finished task")
    }
}
```

- Используем **точное имя метода**, учитываем `_` для Objective-C параметров.
    

---

#### 2️⃣ Обозначить метод `@objc`

```swift
class Processor: NSObject, MyDelegate {
    @objc func processData(_ data: String) {  // ✅ теперь совпадает с @objc протоколом
        print(data)
    }
}
```

- Для optional методов из Objective-C **@objc обязателен** в Swift.
    

---

#### 3️⃣ Проверить типы параметров и возвращаемого значения

- Типы должны совпадать точно, включая **Optional/Non-Optional** и коллекции.
    

```swift
@objc func didFinishTask(_ taskId: Int)  // Int, не Int?
```

---

### Резюме

- Предупреждение появляется при **почти совпадающем методе с optional требованием протокола**.
    
- Исправляется точным совпадением **имени, сигнатуры и @objc аннотаций**.
    
- Помогает корректно реализовать протоколы и избежать рантайм проблем при вызове методов через Objective-C.
    

---
