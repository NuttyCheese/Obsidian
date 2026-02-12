#system_design
## Определение

**Платежные системы** в [[iOS]]-приложениях позволяют пользователям **совершать безопасные покупки** прямо внутри приложения.

Основные задачи:

- Поддержка безопасных и удобных платежей.
    
- Интеграция с банковскими картами, электронными кошельками и подписками.
    
- Соответствие требованиям безопасности (PCI DSS, токенизация).
    

---

## 1. Apple Pay

### Основные возможности

- Быстрые и безопасные платежи с использованием **[[FaceID]] / [[TouchID]]**.
    
- Токенизация платежных данных — реальные номера карт не хранятся в приложении.
    
- Поддержка **In-App Purchases** и **e-commerce**.
    
- Интеграция с Wallet для хранения карт и билетов.
    

### Пример интеграции

```swift
import PassKit

// Проверка доступности Apple Pay
if PKPaymentAuthorizationViewController.canMakePayments(usingNetworks: [.visa, .masterCard]) {
    let paymentRequest = PKPaymentRequest()
    paymentRequest.merchantIdentifier = "merchant.com.myapp"
    paymentRequest.supportedNetworks = [.visa, .masterCard]
    paymentRequest.merchantCapabilities = .capability3DS
    paymentRequest.countryCode = "RU"
    paymentRequest.currencyCode = "RUB"
    
    let paymentSummaryItem = PKPaymentSummaryItem(label: "Товар", amount: NSDecimalNumber(string: "9.99"))
    paymentRequest.paymentSummaryItems = [paymentSummaryItem]
    
    let paymentVC = PKPaymentAuthorizationViewController(paymentRequest: paymentRequest)
    paymentVC?.delegate = self
    present(paymentVC!, animated: true)
}
```

---

## 2. Сторонние SDK

|SDK / Сервис|Описание|
|---|---|
|**Stripe**|Популярный платежный SDK, поддерживает карты, Apple Pay, Google Pay.|
|**PayPal**|Интеграция с PayPal аккаунтами и картами, готовые UI-компоненты.|
|**Adyen**|Глобальная платёжная платформа для e-commerce, поддержка разных методов оплаты.|
|**Qiwi / YooMoney / Tinkoff**|Локальные платежные системы для России и СНГ.|

### Пример интеграции Stripe ([[Swift]])

```swift
import Stripe

// Создание PaymentIntent на сервере и получение clientSecret
let paymentIntentClientSecret = "client_secret_from_server"

let paymentSheet = PaymentSheet(paymentIntentClientSecret: paymentIntentClientSecret)
paymentSheet.present(from: self) { paymentResult in
    switch paymentResult {
    case .completed:
        print("Оплата успешна!")
    case .canceled:
        print("Пользователь отменил оплату")
    case .failed(let error):
        print("Ошибка оплаты: \(error.localizedDescription)")
    }
}
```

---

## 3. Best Practices

1. **Токенизация платежей**
    
    - Никогда не храните данные карты на устройстве.
        
    - Используйте токены и безопасное соединение ([[HTTPS]]/TLS).
        
2. **UX и безопасность**
    
    - Интеграция Apple Pay повышает доверие пользователей.
        
    - Поддержка биометрии для подтверждения оплаты.
        
3. **Обработка ошибок**
    
    - Обрабатывать отмену пользователем, ошибки сети, отказ банка.
        
4. **Локализация и валюта**
    
    - Поддержка нескольких валют и региональных методов оплаты.
        
5. **Регулирование**
    
    - Соответствие стандартам PCI DSS, GDPR и локальному законодательству.
        

---

## 4. Итог

- **Apple Pay** → встроенный безопасный метод оплаты с удобной UX для пользователей iOS.
    
- **Сторонние SDK** → Stripe, PayPal, Adyen и локальные платежные системы расширяют возможности приложения.
    
- **Правильная интеграция + токенизация + TLS** обеспечивает безопасность платежей.
    
- Комбинация **Apple Pay и сторонних SDK** позволяет покрыть широкую аудиторию и региональные особенности платежей.
    

---
