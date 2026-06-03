# Потоковая обработка данных

## Назначение Kafka

Kafka принимает высокочастотные события RTB, кликов, показов и биллинга.
Асинхронный event stream разгружает базу PostgreSQL, позволяет независимо развивать
статистику, аналитику и финансовую сверку.

## Топики

| Topic | Producer | Consumers | Retention |
| --- | --- | --- | --- |
| `rtb.bid_requests` | Bidding Service | Statistics, Analytics, SLA monitoring | 7 дней |
| `rtb.bid_responses` | Bidding Service | Statistics, Analytics, Finance | 30 дней |
| `ad.impressions` | Delivery Service | Statistics, Analytics, Finance | 30 дней |
| `ad.clicks` | Delivery Service | Statistics, Analytics, Finance | 30 дней |
| `campaign.changes` | Campaign Service | Bidding cache warmer, Analytics | 14 дней |
| `finance.billing_events` | Finance Service / Statistics | Finance, Analytics | 90 дней |
| `dlq.events` | Consumers | Support/engineering replay tools | 14 дней |

## Схема событий

Базовые поля всех событий:

- `event_id`;
- `event_type`;
- `schema_version`;
- `occurred_at`;
- `producer`;
- `trace_id`;
- `partner_id`;
- `campaign_id`, если применимо.

Пример события click/impression:

```json
{
  "event_id": "evt-123",
  "event_type": "ad.impression",
  "schema_version": "1.0",
  "occurred_at": "2026-06-01T10:00:00Z",
  "request_id": "bid-456",
  "partner_id": "dsp-a",
  "campaign_id": "cmp-789",
  "creative_id": "crt-001",
  "price_micros": 12000
}
```

## Consumer groups

| Consumer group | Назначение |
| --- | --- |
| `statistics-aggregator` | Оперативные счетчики кликов, показов, win/loss |
| `analytics-loader` | Загрузка событий в ClickHouse |
| `finance-billing` | Списание бюджета по billable events |
| `bidding-cache-updater` | Обновление Redis после изменений кампаний и бюджетов |
| `sla-monitoring` | Метрики DSP latency, errors и timeouts |

