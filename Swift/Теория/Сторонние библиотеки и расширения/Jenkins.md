#third-party_library #automation 
## 📘 Определение

**Jenkins** — это **система автоматизации ([[CI]]/[[CD]])** с открытым исходным кодом, которая позволяет автоматически **собирать, тестировать и деплоить проекты**.  
Часто используется в [[iOS]]-разработке для автоматической сборки [[Xcode]]-проектов, запуска юнит-тестов, генерации артефактов (IPA) и доставки на [[TestFlight]] или App Store.

---

## 🔹 Примеры использования

### 1. Установка Jenkins (macOS, Homebrew)

```bash
brew install jenkins-lts
brew services start jenkins-lts
```

---

### 2. Настройка Jenkins Job для iOS-проекта

1. Создать новый **Freestyle project** или **Pipeline**.
    
2. В разделе **Source Code Management** указать репозиторий Git.
    
3. В разделе **Build** добавить скрипт:
    

```bash
#!/bin/bash
cd MyApp
xcodebuild -scheme "MyApp" -sdk iphoneos -configuration Release clean build
```

---

### 3. Pipeline для сборки и тестов (Jenkinsfile)

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/username/MyApp.git'
            }
        }
        stage('Build') {
            steps {
                sh 'xcodebuild -scheme MyApp -sdk iphonesimulator -destination "platform=iOS Simulator,name=iPhone 14" clean build'
            }
        }
        stage('Test') {
            steps {
                sh 'xcodebuild test -scheme MyApp -sdk iphonesimulator -destination "platform=iOS Simulator,name=iPhone 14"'
            }
        }
    }
}
```

---

### 4. Отправка артефакта на TestFlight

```groovy
stage('Upload to TestFlight') {
    steps {
        sh 'fastlane beta'
    }
}
```

---

### 5. Настройка уведомлений после сборки

```groovy
post {
    success {
        slackSend(channel: '#ios-builds', message: "Сборка прошла успешно ✅")
    }
    failure {
        slackSend(channel: '#ios-builds', message: "Сборка провалилась ❌")
    }
}
```
