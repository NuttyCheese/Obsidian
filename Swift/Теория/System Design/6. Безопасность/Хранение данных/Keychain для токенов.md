#system_design
## Определение

**Keychain** — это **безопасное хранилище** в [[iOS]] для конфиденциальных данных, таких как:

- пароли
    
- токены (access token, refresh token)
    
- приватные ключи
    
- сертификаты
    

Цель: **защитить критические данные от несанкционированного доступа**, даже если устройство взломано.

---

## 1. Особенности [[KeyChain]]

|Особенность|Описание|
|---|---|
|Безопасное хранилище|Данные шифруются аппаратно и системой iOS|
|Persistent|Данные сохраняются между установками (если настроено)|
|Доступ по правам|Можно ограничить доступ по условиям: когда устройство разблокировано, при биометрии и т.д.|
|Интеграция с Secure Enclave|Для хранения криптографических ключей и биометрии|

---

## 2. Основные элементы

- **Service** → логическая категория, например, `"com.myapp.accessToken"`.
    
- **Account** → идентификатор пользователя или ресурса.
    
- **Data** → зашифрованные данные (токен, пароль).
    
- **Access Control** → условия доступа: при разблокировке, при [[FaceID]]/[[TouchID]].
    

---

## 3. Пример хранения токена в [[Swift]]

```swift
import Security

func saveToken(_ token: String, service: String, account: String) -> Bool {
    guard let data = token.data(using: .utf8) else { return false }
    
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrService as String: service,
        kSecAttrAccount as String: account,
        kSecValueData as String: data,
        kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlocked
    ]
    
    SecItemDelete(query as CFDictionary) // удалить старый токен
    let status = SecItemAdd(query as CFDictionary, nil)
    
    return status == errSecSuccess
}

func loadToken(service: String, account: String) -> String? {
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrService as String: service,
        kSecAttrAccount as String: account,
        kSecReturnData as String: true,
        kSecMatchLimit as String: kSecMatchLimitOne
    ]
    
    var dataTypeRef: AnyObject?
    let status = SecItemCopyMatching(query as CFDictionary, &dataTypeRef)
    
    guard status == errSecSuccess, let data = dataTypeRef as? Data else { return nil }
    return String(data: data, encoding: .utf8)
}
```

---

## 4. Особенности доступа

|Атрибут|Описание|
|---|---|
|`kSecAttrAccessibleWhenUnlocked`|Доступ только при разблокированном устройстве|
|`kSecAttrAccessibleAfterFirstUnlock`|Доступ даже если устройство заблокировано, после первой разблокировки|
|`kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly`|Доступ только при установленном пароле, не переносится на другие устройства|

---

## 5. Best Practices для токенов

1. **Использовать Keychain для всех токенов** → никогда не хранить их в UserDefaults.
    
2. **Access Control** → ограничивать доступ токенов с помощью `kSecAttrAccessibleWhenUnlocked` или биометрии.
    
3. **Secure Enclave** → при необходимости хранить приватные ключи или refresh token.
    
4. **Удаление токенов при логаут** → очищать Keychain при выходе пользователя.
    
5. **Шифрование данных** → Keychain шифрует данные, но при желании можно добавить дополнительный слой шифрования.
    

---

## 6. Итог

- **Keychain** — надежное хранилище для токенов и чувствительных данных.
    
- Используется совместно с **Secure Enclave** и биометрией для повышения безопасности.
    
- **Правильная настройка доступа и очистка токенов** предотвращает утечки и компрометацию.
    
- В сочетании с **TLS/[[HTTPS]], OAuth и JWT** Keychain создаёт безопасный стек аутентификации для iOS-приложений.
    

---
