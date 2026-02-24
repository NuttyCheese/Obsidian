**PostgreSQL** (часто называют просто **Postgres**) — это одна из самых мощных, надёжных и популярных **открытых реляционных СУБД** в мире.

По состоянию на февраль 2026 года PostgreSQL остаётся **лидером** среди реляционных баз данных по функциональности, расширяемости и качеству сообщества. Многие разработчики и компании предпочитают Postgres вместо [[MySQL]], особенно в проектах, где важны:

- сложные запросы  
- строгая целостность данных  
- расширяемость (JSONB, PostGIS, TimescaleDB, pgvector и т.д.)  
- ACID-транзакции  
- высокая надёжность и отказоустойчивость

### Ключевые версии и актуальное состояние (2026)

| Версия       | Дата выхода       | Основные фичи (2024–2026)                              | Статус поддержки в 2026 | Рекомендация для новых проектов |
|--------------|-------------------|---------------------------------------------------------|--------------------------|---------------------------------|
| **PostgreSQL 16** | сентябрь 2023     | Улучшенный параллелизм, логическая репликация, JSONB   | Полная поддержка до 2028 | Стабильный выбор                |
| **PostgreSQL 17** | сентябрь 2024     | Улучшенный MERGE, incremental backup, лучшее партиционирование | Полная поддержка до 2029 | Рекомендуется для продакшена    |
| **PostgreSQL 18** (beta / RC) | ожидается 2025–2026 | Vector search (pgvector встроен), улучшенный AI/ML, Iceberg tables | Innovation release       | Для экспериментов и новых фич   |
| **TimescaleDB** (расширение) | 2023–2026         | Time-series + compression + continuous aggregates      | Активно развивается      | Метрики, аналитика, IoT         |
| **pgvector** (расширение)   | 2023–2026         | Векторный поиск (HNSW, IVFFlat)                        | Активно развивается      | Семантический поиск, RAG, ML    |

### Самые популярные сценарии использования PostgreSQL в iOS/macOS-приложениях 2026

| Сценарий                             | Почему PostgreSQL выигрывает у MySQL / других              | Самый популярный стек 2026                    |
| ------------------------------------ | ---------------------------------------------------------- | --------------------------------------------- |
| **Бэкенд для мобильных приложений**  | JSONB + GIN-индексы, сложные JOIN'ы, CTE, Window functions | [[Vapor]] / Hummingbird + Fluent + PostgreSQL |
| **Оффлайн-first + синхронизация**    | Supabase (PostgreSQL + Realtime) — лучший аналог Firebase  | Supabase (Swift SDK)                          |
| **Временные ряды / аналитика**       | TimescaleDB (расширение) — лидер рынка time-series         | TimescaleDB + Grafana                         |
| **Семантический поиск / RAG / ML**   | pgvector — встроенный векторный поиск (HNSW)               | pgvector + OpenAI / Grok embeddings           |
| **Геолокация / карты**               | PostGIS (расширение) — золотой стандарт гео-БД             | PostGIS + Mapbox / Apple Maps                 |
| **Финансы / транзакции**             | Полная ACID, CHECK-констрейнты, точные DECIMAL             | PostgreSQL + Citus (шардирование)             |
| **Локальное хранение на устройстве** | PostgreSQL **не подходит** (слишком тяжёлый)               | [[SQLite]] / [[SwiftData]] / [[Core Data]]    |

### Самый популярный стек для Swift-приложений 2026

1. **Бэкенд** — [[Vapor]] / Hummingbird + FluentPostgreSQL  
2. **Облако** — Supabase (PostgreSQL + Realtime + Auth + Storage)  
3. **Локально** — SwiftData / [[Core Data]] / GRDB ([[SQLite]])  
4. **Расширения** — pgvector (векторный поиск), PostGIS (гео), TimescaleDB (time-series)  
5. **Миграции** — Fluent Migrations / Flyway / Liquibase  
6. **ORM** — Fluent (Vapor), GRDB (локально), SQLKit (низкоуровневый)

### Пример подключения к PostgreSQL из Swift (Vapor 4+)

```swift
import Vapor
import FluentPostgreSQLDriver

public func configure(_ app: Application) async throws {
    app.databases.use(.postgres(
        hostname: Environment.get("DB_HOST") ?? "localhost",
        port: Environment.get("DB_PORT").flatMap(Int.init) ?? 5432,
        username: Environment.get("DB_USER") ?? "postgres",
        password: Environment.get("DB_PASSWORD") ?? "",
        database: Environment.get("DB_NAME") ?? "myapp"
    ), as: .psql)
    
    app.migrations.add(CreateUsersTable())
    
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

### Лучшие практики PostgreSQL в Swift-приложениях 2026

- **PostgreSQL 17** — основной выбор для новых проектов  
- **utf8mb4** (или лучше `utf8mb4_unicode_ci`) — кодировка по умолчанию  
- **JSONB** — храни сложные структуры в JSONB с GIN-индексами  
- **pgvector** — для семантического поиска и RAG  
- **TimescaleDB** — для временных рядов и метрик  
- **Supabase** — если нужен быстрый старт с реал-тайм и аутентификацией  
- **Swift 6 strict concurrency** — FluentPostgreSQLDriver полностью поддерживает async/await и Sendable  
- **Миграции** — Fluent Migrations или Flyway  
- **Тестирование** — Testcontainers (PostgreSQL в Docker) или SQLite для unit-тестов  
- **Документируйте** — пиши комментарий «PostgreSQL 17 + pgvector — семантический поиск»

**Короткий девиз 2026**:
> «PostgreSQL в 2026 году — это когда тебе нужна самая мощная, расширяемая и надёжная реляционная БД с открытым исходным кодом.  
> Для мобильных бэкендов — это часто лучший выбор (особенно с Supabase или Vapor + Fluent).  
> Если нужна максимальная простота и реал-тайм — Supabase, если максимальная производительность и расширяемость — чистый PostgreSQL 17 + pgvector + TimescaleDB.»
