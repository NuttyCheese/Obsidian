**AFNetworking** — это популярная сторонняя сетевая библиотека для [[iOS]] и macOS, написанная на [[Objective-C]], предоставляющая удобный высокоуровневый [[API]] для выполнения [[HTTP]]-запросов, загрузки/отправки данных, работы с [[JSON]], очередями операций и мониторинга сети.  
Хотя Core API на Objective-C, библиотека долгое время использовалась и в [[Swift]]-проектах до появления [[Alamofire]].

### **К какому стеку относится**

- **Сторонний фреймворк (3rd-party)**
    
- Ближе всего по назначению к **[[Foundation]] → [[URLSession]]**, но не является частью стандартных фреймворков.
    
- Часто использовался в **[[UIKit]]-проектах** до Swift/Alamofire.
    

---

## **1. Простейший GET-запрос (Objective-C)**

```objective-c
AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];

[manager GET:@"https://api.example.com/users"
  parameters:nil
     headers:nil
    progress:nil
     success:^(NSURLSessionDataTask *task, id responseObject) {
        NSLog(@"Ответ: %@", responseObject);
     }
     failure:^(NSURLSessionDataTask *task, NSError *error) {
        NSLog(@"Ошибка: %@", error.localizedDescription);
     }];
```

**Комментарий:** базовый пример получения JSON.

---

## **2. [[POST-HTTP|POST]]-запрос с параметрами**

```objective-c
NSDictionary *params = @{@"email": @"test@mail.com",
                         @"password": @"123456"};

AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
manager.requestSerializer = [AFJSONRequestSerializer serializer];
manager.responseSerializer = [AFJSONResponseSerializer serializer];

[manager POST:@"https://api.example.com/login"
   parameters:params
      headers:nil
     progress:nil
      success:^(NSURLSessionDataTask *task, id responseObject) {
          NSLog(@"Успешно! %@", responseObject);
      }
      failure:^(NSURLSessionDataTask *task, NSError *error) {
          NSLog(@"Ошибка: %@", error.localizedDescription);
      }];
```

**Комментарий:** отправляем JSON-тело, сериализация в JSON.

---

## **3. Загрузка файла**

```objective-c
AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];

NSURL *fileURL = [NSURL fileURLWithPath:@"/path/to/file.png"];

[manager POST:@"https://api.example.com/upload"
   parameters:nil
      headers:nil
constructingBodyWithBlock:^(id<AFMultipartFormData> formData) {
    [formData appendPartWithFileURL:fileURL
                               name:@"file"
                           fileName:@"file.png"
                           mimeType:@"image/png"
                              error:nil];
}
      progress:^(NSProgress *uploadProgress) {
          NSLog(@"Прогресс: %f", uploadProgress.fractionCompleted);
      }
       success:^(NSURLSessionDataTask *task, id responseObject) {
           NSLog(@"Готово %@", responseObject);
       }
       failure:^(NSURLSessionDataTask *task, NSError *error) {
           NSLog(@"Ошибка %@", error.localizedDescription);
       }];
```

**Комментарий:** multipart-загрузка файла.

---

## **4. Мониторинг сети (Reachability)**

```objective-c
AFNetworkReachabilityManager *reachability = [AFNetworkReachabilityManager sharedManager];

[reachability setReachabilityStatusChangeBlock:^(AFNetworkReachabilityStatus status) {
    switch (status) {
        case AFNetworkReachabilityStatusNotReachable:
            NSLog(@"Нет сети");
            break;
        case AFNetworkReachabilityStatusReachableViaWiFi:
            NSLog(@"Wi-Fi");
            break;
        case AFNetworkReachabilityStatusReachableViaWWAN:
            NSLog(@"Cellular");
            break;
        default:
            NSLog(@"Неизвестно");
            break;
    }
}];

[reachability startMonitoring];
```

**Комментарий:** отслеживание изменений сети.

---

## **5. Параллельные операции через AFHTTPRequestOperationManager**

```objective-c
AFHTTPRequestOperationManager *manager = [AFHTTPRequestOperationManager manager];

NSArray *urls = @[@"https://api.example.com/1",
                  @"https://api.example.com/2"];

NSOperationQueue *queue = manager.operationQueue;

for (NSString *url in urls) {
    AFHTTPRequestOperation *operation = [manager GET:url
                                          parameters:nil
                                             success:^(AFHTTPRequestOperation *operation, id responseObject) {
                                                 NSLog(@"Ответ для %@: %@", url, responseObject);
                                             }
                                             failure:^(AFHTTPRequestOperation *operation, NSError *error) {
                                                 NSLog(@"Ошибка: %@", error);
                                             }];
    [queue addOperation:operation];
}
```

**Комментарий:** пример работы с операциями (до появления Task API).

---

## **6. Swift-использование (через bridging header)**

```swift
let manager = AFHTTPSessionManager()
manager.responseSerializer = AFJSONResponseSerializer()

manager.get("https://api.example.com/users",
            parameters: nil,
            headers: nil,
            progress: nil,
            success: { _, response in
                print("Ответ: \(response ?? "")")
            },
            failure: { _, error in
                print("Ошибка: \(error.localizedDescription)")
            })
```

**Комментарий:** типичный Swift-обёрнутый вызов, если проект старый и использует Objective-C библиотеку.
