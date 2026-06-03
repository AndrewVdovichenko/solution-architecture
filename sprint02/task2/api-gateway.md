# API Gateway для DSP-интеграции

## Назначение

API Gateway отделяет внешнюю DSP-интеграцию от внутренних сервисов AdScale. Он
принимает OpenRTB-запросы, применяет защитные политики и направляет запросы в
Bidding Service.

Рекомендуемые технологии: Envoy или NGINX.

Так как в команде нет выделенного SRE, предпочтительно использовать минимально настраиваемый инструмент.

## Основные функции

| Функция | Описание |
| --- | --- |
| Authentication | API key для DSP-партнеров |
| Circuit breaker | Быстрое размыкание при деградации Bidding Service |
| Observability | Метрики: latency, error rate, timeout, rejected requests |
| Rate limiting | Ограничения RPS на партнера и глобальный лимит |
| Request normalization | Приведение OpenRTB запроса к внутреннему контракту |
| Routing | Маршрутизация по partner_id, endpoint, версии API |
| Timeout control | Единый таймаут для downstream-вызова Bidding Service |

## Маршрутизация

Пример маршрутов:

| Endpoint | Назначение |
| --- | --- |
| `POST /openrtb/2.5/bid` | Основной bid endpoint |
| `POST /openrtb/2.5/bid/test` | Тестовый bid endpoint |
| `GET /health` | Health check |

Для canary-релизов API gateway должен поддерживать маршруты с весами на разные версии
Bidding Service.

## Rate limiting

Политики:

- лимит RPS на partner_id;
- приоритет текущих клиентов при деградации;
- возврат no-bid или 429 для превышения квоты по договоренности с DSP.

## Timeout budget

Gateway задает таймаут на весь downstream-вызов. Если DSP требует стабильный
ответ до 80 ms, внутренний таймаут до Bidding Service должен быть меньше внешнего
лимита, например 60-70 ms, чтобы оставить запас на сеть и сериализацию.

## Circuit breaker

Размыкатель включается при:

- росте timeout rate;
- превышении error rate;
- заполнении connection pool;
- росте задержек P95/P99 к Bidding Service.

В открытом состоянии gateway возвращает быстрый no-bid/default response для
защиты платформы от каскадной деградации.

## Мониторинг

Обязательные метрики:

- RPS по partner_id;
- P50/P95/P99 latency;
- timeout rate;
- error rate;
- rejected by rate limit;
- circuit breaker state;
- доля bid/no-bid;

Метрики должны быть доступны для технической команды и использоваться
для SLA-отчетности.
