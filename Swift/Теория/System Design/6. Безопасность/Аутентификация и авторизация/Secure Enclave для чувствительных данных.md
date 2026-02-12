#system_design
## Определение

**Secure Enclave** — это отдельный аппаратный модуль на [[iOS]]-устройствах, предназначенный для **безопасного хранения и обработки чувствительных данных**.

Цель: **обеспечить высокий уровень безопасности**, минимизируя риск кражи или компрометации данных даже при взломе основной системы.

---

## 1. Основные возможности

- Хранение **криптографических ключей** (для шифрования, подписи и аутентификации).
    
- Генерация и хранение **Private key**, который **никогда не покидает Secure Enclave**.
    
- Поддержка **biometric authentication** ([[FaceID]]/[[TouchID]]) для доступа к данным.
    
- Защита от атак уровня ОС и физических атак на устройство.
    

---

## 2. Принцип работы

1. **Ключи генерируются внутри Secure Enclave** и **не покидают чип**.
    
2. **Приложение взаимодействует через API**, а не напрямую с ключом.
    
3. **Подтверждение пользователя через биометрию** (FaceID/TouchID) выполняется внутри Secure Enclave.
    
4. **Данные шифруются/дешифруются** с использованием ключей Enclave, исключая доступ извне.
    

> Важная особенность: даже если устройство взломано, приватные ключи остаются защищёнными.

---

## 3. Интеграция в iOS

### Создание ключа в Secure Enclave

```swift
import Security

let access = SecAccessControlCreateWithFlags(nil,
                                             kSecAttrAccessibleWhenUnlockedThisDeviceOnly,
                                             .privateKeyUsage,
                                             nil)

let attributes: [String: Any] = [
    kSecAttrKeyType as String: kSecAttrKeyTypeECSECPrimeRandom,
    kSecAttrKeySizeInBits as String: 256,
    kSecAttrTokenID as String: kSecAttrTokenIDSecureEnclave,
    kSecPrivateKeyAttrs as String: [
        kSecAttrIsPermanent as String: true,
        kSecAttrAccessControl as String: access!
    ]
]

var error: Unmanaged<CFError>?
guard let privateKey = SecKeyCreateRandomKey(attributes as CFDictionary, &error) else {
    fatalError("Не удалось создать ключ: \(error!.takeRetainedValue() as Error)")
}
```

### Использование ключа для подписи данных

```swift
let message = "Sensitive data".data(using: .utf8)!
var error: Unmanaged<CFError>?

guard let signature = SecKeyCreateSignature(privateKey,
                                            .ecdsaSignatureMessageX962SHA256,
                                            message as CFData,
                                            &error) else {
    fatalError("Не удалось подписать сообщение: \(error!.takeRetainedValue() as Error)")
}
```

---

## 4. Применение

- **Biometric authentication** → FaceID / TouchID с безопасным хранением ключей.
    
- **Хранение токенов и паролей** → вместе с [[Keychain]] и доступом через Secure Enclave.
    
- **Подпись транзакций** → финансовые приложения.
    
- **Шифрование локальных данных** → защита конфиденциальной информации.
    

---

## 5. Best Practices

1. **Использовать Secure Enclave для приватных ключей** → никогда не хранить их в обычной памяти.
    
2. **Использовать Keychain совместно с Enclave** → для безопасного хранения токенов.
    
3. **Применять биометрическую проверку** → повышает уровень безопасности.
    
4. **Ограничивать доступ ключей по времени и политике** → `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`.
    
5. **Не хранить чувствительные данные в виде открытого текста** → только зашифрованные.
    
6. **Обрабатывать ошибки и fallback** → в случае невозможности доступа к Enclave.
    

---

## 6. Преимущества Secure Enclave

|Преимущество|Описание|
|---|---|
|Аппаратная защита|Отделён от основной CPU, защищён от взлома ОС|
|Безопасное хранение ключей|Private key не покидает Enclave|
|Biometric integration|Поддержка FaceID/TouchID для доступа к ключам|
|Криптография на устройстве|Подпись, шифрование и проверка данных локально|
|Минимизация рисков|Даже при компрометации iOS ключи остаются защищёнными|

---

## Итог

- **Secure Enclave** — аппаратный модуль для безопасного хранения и обработки данных.
    
- Используется для **ключей, биометрии, подписи и шифрования**.
    
- Важная практика iOS-разработки: **комбинировать Secure Enclave с Keychain и биометрией для защиты чувствительных данных**.
    
- Обеспечивает высокий уровень безопасности без компромиссов для UX.
    

---
