**MySQL** — это одна из самых популярных и широко используемых систем управления реляционными базами данных (СУБД) с открытым исходным кодом.

По состоянию на февраль 2026 года MySQL остаётся **одним из лидеров рынка** (вместе с PostgreSQL, MariaDB и [[SQLite]]) и продолжает активно развиваться под управлением Oracle (основная ветка) и сообществом (MariaDB как форк).

### Ключевые версии и актуальное состояние (2026)

| Версия             | Дата выхода         | Основные фичи 2025–2026                                    | Статус поддержки в 2026    | Рекомендация для новых проектов |
| ------------------ | ------------------- | ---------------------------------------------------------- | -------------------------- | ------------------------------- |
| **MySQL 8.0**      | апрель 2018         | [[JSON]], CTE, Window functions, Roles, Histograms         | Полная поддержка до 2026+  | Основной стабильный выбор       |
| **MySQL 8.4 LTS**  | апрель 2024         | Улучшенный JSON, Group Replication, Heatwave ML            | Долгосрочная (LTS) до 2032 | Рекомендуется для продакшена    |
| **MySQL 9.0**      | ожидается 2025–2026 | Vector Search (Heatwave), улучшенный Optimizer, AI-индексы | Innovation release         | Для экспериментов и новых фич   |
| **MariaDB 11.x**   | 2023–2025           | ColumnStore, Dynamic Columns, System-versioned tables      | Активно развивается        | Альтернатива Oracle-MySQL       |
| **MySQL Heatwave** | 2021–2026           | In-memory query accelerator + ML на GPU/CPU                | Облачный сервис Oracle     | Для аналитики и ML-запросов     |

### Самые популярные сценарии использования [[MySQL]] в [[iOS]]/macOS-приложениях 2026

1. **Локальная БД на устройстве** (через SQLite, а не чистый MySQL)  
   → MySQL **не предназначен** для работы на клиенте (iOS/watchOS)  
   → Вместо него используют **SQLite** (встроен в iOS) или **[[SwiftData]]** / **[[Core Data]]**

2. **Бэкенд для мобильных приложений**  
   → MySQL как основная БД сервера (API, [[REST]]/[[GraphQL]])  
   → Самый частый стек: MySQL 8.4 + Node.js / Python (FastAPI) / Go / [[Vapor]] / Laravel / Spring Boot

3. **Облачные решения**  
   - AWS RDS for MySQL / Aurora MySQL  
   - Google Cloud SQL for MySQL  
   - Azure Database for MySQL  
   - Oracle MySQL Heatwave (встроенный ML и аналитика)  
   - PlanetScale (serverless MySQL с Vitess)  
   - Aiven / DigitalOcean Managed Databases

### Самый актуальный пример подключения к MySQL из Swift (Vapor 4+ / Swift 2026)

```swift
import Vapor
import FluentMySQLDriver

// configure.swift
public func configure(_ app: Application) async throws {
    app.databases.use(.mysql(
        hostname: Environment.get("DATABASE_HOST") ?? "localhost",
        port: Environment.get("DATABASE_PORT").flatMap(Int.init) ?? 3306,
        username: Environment.get("DATABASE_USER") ?? "root",
        password: Environment.get("DATABASE_PASSWORD") ?? "",
        database: Environment.get("DATABASE_NAME") ?? "myapp"
    ), as: .mysql)
    
    app.migrations.add(CreateUsersTable())
    
    // Запуск миграций
    try await app.autoMigrate()
}

// Модель
final class User: Model {
    static let schema = "users"
    
    @ID(key: .id)
    var id: UUID?
    
    @Field(key: "name")
    var name: String
    
    @Field(key: "email")
    var email: String
    
    @Timestamp(key: "created_at", on: .create)
    var createdAt: Date?
    
    init() {}
    
    init(id: UUID? = nil, name: String, email: String) {
        self.id = id
        self.name = name
        self.email = email
    }
}
```

### Лучшие практики работы с MySQL в Swift-приложениях 2026

- **MySQL 8.4 LTS** — основной выбор для продакшена (до 2032 года)  
- **utf8mb4_unicode_ci** — кодировка и collation по умолчанию (полная поддержка эмодзи)  
- **InnoDB** — единственный движок, который стоит использовать (поддержка транзакций, foreign keys)  
- **Prepared statements** — всегда используй (защита от SQL-инъекций) — FluentMySQLDriver делает это автоматически  
- **Connection pooling** — обязательно (Vapor / Hummingbird / Kitura используют пул по умолчанию)  
- **Read replicas** — для высокой нагрузки на чтение (AWS Aurora, PlanetScale)  
- **JSON колонка** — активно используй (MySQL 8.0+ поддерживает [[JSON]] с индексами)  
- **Heatwave / ML-функции** — для встроенной аналитики и предсказаний прямо в БД  
- **Swift 6 strict concurrency** — FluentMySQLDriver полностью поддерживает [[async]]/[[await]] и [[Sendable]]  
- **Миграции** — используй Fluent Migrations или Flyway / Liquibase для крупных проектов  
- **Тестирование** — in-memory MySQL (Testcontainers) или SQLite для unit-тестов

**Короткий девиз 2026**:
> «MySQL в 2026 году — это когда тебе нужна надёжная, проверенная временем реляционная БД с отличной производительностью и огромным сообществом.  
> Для большинства мобильных бэкендов — это всё ещё один из лучших выборов (особенно с MySQL 8.4 LTS и Heatwave).  
> Если нужна максимальная совместимость со Swift — смотри на Vapor + FluentMySQLDriver.»
