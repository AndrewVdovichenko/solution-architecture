# Задание 3. Данные, масштабирование и отказоустойчивость

В разделе описана стратегия работы с данными для микросервисной архитектуры
AdScale: Database per Service, масштабирование БД, кеширование, Kafka и отказоустойчивость.

## Состав артефактов

- [database-strategy.md](database-strategy.md) — выбор базы данных для каждого сервиса.
- [scaling.md](scaling.md) — стратегии репликации и шардирования.
- [caching.md](caching.md) — дизайн кэширования.
- [event-streaming.md](event-streaming.md) — архитектура Kafka.
- [failover.md](failover.md) — отказоустойчивость данных.
- [diagrams/data-architecture.puml](diagrams/data-architecture.puml) — диаграмма
  данных.

## Ключевая идея

OLTP, RTB hot data, event stream и аналитика должны быть разделены. PostgreSQL
остается для транзакционных доменов, Redis обслуживает быстрые чтения RTB,
Kafka принимает поток событий, а ClickHouse используется для аналитики и
агрегаций.
