# Bidding Service

## Назначение

Bidding Service — первый сервис, выделяемый из монолита AdScale. Он отвечает за
обработку RTB-запросов, выбор ставки и формирование bid response для DSP.

Сервис находится в критическом пути, поэтому его дизайн оптимизирован под
предсказуемую задержку и горизонтальное масштабирование.

## Границы ответственности

Сервис делает:

- принимает нормализованный bid request от API Gateway;
- валидирует минимально необходимые поля запроса;
- подбирает подходящие кампании и ставки из кеша Redis;
- применяет бизнес-правила аукциона;
- формирует bid response или no-bid response;
- публикует bid events в Kafka;
- пишет технические логи в собственную БД вне критического пути.

Сервис не делает:

- управление кампаниями;
- финансовые транзакции в синхронном пути;
- тяжелую аналитику;
- прямую запись кликов и показов в OLTP-БД.

## API

### Внешний контракт

Внешний контракт с DSP остается OpenRTB over HTTPS. API Gateway принимает
запросы DSP и нормализует их для Bidding Service.

### Внутренний API

Для взаимодействия API Gateway -> Bidding Service используется gRPC, так как
этот участок чувствителен к задержкам, требует строгого контракта.

Пример логического контракта:

```protobuf
service BiddingService {
  rpc GetBid(BidRequest) returns (BidResponse);
}

message BidRequest {
  string request_id = 1;
  string partner_id = 2;
  string user_id = 3;
  string placement_id = 4;
  string device_type = 5;
  repeated string segments = 6;
  int64 timeout_ms = 7;
}

message BidResponse {
  string request_id = 1;
  bool bid = 2;
  string campaign_id = 3;
  string creative_id = 4;
  int64 price_micros = 5;
  string reason = 6;
}
```

## Зависимости

| Зависимость | Тип | Использование |
| --- | --- | --- |
| Redis Cluster | Sync read | Кампании, ставки, таргетинг, бюджет |
| Kafka | Async write | bid request, bid response, no_bid, win/loss |
| Campaign Service | Async/sync admin | Публикация изменений кампаний |
| Finance Service | Async/sync limited | Снапшот бюджета, финансовые события |
| Bidding DB | SQL вне критического пути | Конфигурация, аудит |

## Модель данных

### Campaign snapshot в Redis

- `campaign_id`;
- `advertiser_id`;
- `status`;
- `targeting_rules`;
- `bid_strategy`;
- `max_bid`;
- `daily_budget_left`;
- `creative_ids`;
- `updated_at`;
- `version`.

### Bid decision event

- `event_id`;
- `request_id`;
- `partner_id`;
- `campaign_id`;
- `creative_id`;
- `bid_price`;
- `decision`;
- `reason`;
- `latency_ms`;
- `created_at`.

## Latency budget

Ориентир для промежуточного решения:

| Этап | Бюджет |
| --- | --- |
| API Gateway auth/rate limit/routing | 5-10 ms |
| Bidding Service decision | 40-55 ms |
| Redis reads | 5-10 ms |
| Response serialization/network | 5-10 ms |
| Резерв | 10-20 ms |

Синхронные обращения к PostgreSQL в критическом пути запрещены для штатного сценария.

## Масштабирование

- Сервис stateless, масштабируется горизонтально.
- Redis используется как общий горячий кеш.
- Kafka принимает события асинхронно.
- Для релизов используется canary-channel, чтобы не нарушать DSP SLA.
