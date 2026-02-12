#system_design
## Определение

**SSL Pinning** — это техника защиты iOS-приложений от MITM-атак (Man-in-the-Middle), которая **привязывает приложение к конкретному сертификату или публичному ключу сервера**.

Цель: убедиться, что **приложение соединяется только с доверенным сервером**, даже если злоумышленник подменяет сертификаты на уровне сети.

---

## 1. Виды Pinning

|Вид|Описание|Преимущества|Недостатки|
|---|---|---|---|
|**Certificate Pinning**|Привязка к конкретному SSL-сертификату сервера|Простая реализация|Требует обновления сертификата при его смене|
|**Public Key Pinning**|Привязка к публичному ключу сертификата|Не требует обновления при смене сертификата|Сложнее реализовать|

---

## 2. Принцип работы

1. Приложение хранит **сертификат или публичный ключ** локально (в bundle).
    
2. При установлении [[HTTPS]] соединения через **[[URLSession]]** приложение проверяет, совпадает ли сертификат сервера с локальным.
    
3. Если совпадает → соединение разрешено.
    
4. Если не совпадает → соединение разрывается, предотвращая MITM.
    

---

## 3. Реализация Certificate Pinning в [[iOS]]

```swift
import Foundation

class SSLPinningDelegate: NSObject, URLSessionDelegate {
    func urlSession(_ session: URLSession,
                    didReceive challenge: URLAuthenticationChallenge,
                    completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {
        
        guard let serverTrust = challenge.protectionSpace.serverTrust,
              let certificate = SecTrustGetCertificateAtIndex(serverTrust, 0) else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }

        let remoteCertData = SecCertificateCopyData(certificate) as Data
        let localCertPath = Bundle.main.path(forResource: "server", ofType: "cer")!
        let localCertData = try! Data(contentsOf: URL(fileURLWithPath: localCertPath))
        
        if remoteCertData == localCertData {
            let credential = URLCredential(trust: serverTrust)
            completionHandler(.useCredential, credential)
        } else {
            completionHandler(.cancelAuthenticationChallenge, nil)
        }
    }
}
```

**Объяснение:**

- `serverTrust` → сертификат, присланный сервером.
    
- Сравниваем его с локально сохранённым (`server.cer`).
    
- Только совпадение разрешает HTTPS соединение.
    

---

## 4. Реализация Public Key Pinning

- В отличие от certificate pinning, сравнивается **только публичный ключ**.
    
- Позволяет серверу обновлять сертификаты без изменения в приложении.
    
- Используется функция `SecKeyCopyPublicKey` для извлечения публичного ключа из сертификата.
    

Пример:

```swift
let publicKey = SecCertificateCopyKey(certificate)!
let publicKeyData = SecKeyCopyExternalRepresentation(publicKey, nil)! as Data

let localKeyData = ... // данные публичного ключа из bundle

if publicKeyData == localKeyData {
    // доверенное соединение
} else {
    // отклонить соединение
}
```

---

## 5. Best Practices для iOS

1. **Всегда использовать HTTPS + Pinning** → основная защита от MITM.
    
2. **Public Key Pinning предпочтительнее** → меньше проблем с обновлением сертификатов.
    
3. **Хранить сертификаты / ключи в Bundle** → защищённое размещение.
    
4. **Обрабатывать ошибки соединения корректно** → информировать пользователя или fallback на безопасный режим.
    
5. **Регулярно обновлять сертификаты и публичные ключи** при смене на сервере.
    
6. **Комбинировать с другими методами** → TLS 1.3, OAuth/JWT, Secure Enclave.
    

---

## 6. Итог

- **SSL Pinning** → критический механизм для защиты от MITM-атак.
    
- **Certificate Pinning** → проще, но требует обновления сертификатов.
    
- **Public Key Pinning** → гибче, защищает при смене сертификата.
    
- **iOS-инструменты:** [[URLSessionDelegate]], SecTrust, SecCertificate, SecKey.
    
- **Комбинация с TLS, OAuth и Secure Enclave** создаёт безопасный стек для мобильного приложения.
    

---
