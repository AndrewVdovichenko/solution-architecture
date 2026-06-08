# Кеширование

## Назначение

Кеш нужен для снижения задержек в критическом пути. Bidding Service не должен
обращаться к PostgreSQL на каждый bid запрос.

Основное хранилище кеша — Redis.

## Данные для кеширования

| Данные | Ключ | TTL | Инвалидация |
| --- | --- | --- | --- |
| Campaign snapshot | `campaign:{campaign_id}` | 5-15 минут | `campaign.updated` event |
| Active campaigns by placement | `placement:{placement_id}:campaigns` | 1-5 минут | `campaign.updated`, `campaign.paused` |
| Targeting rules | `targeting:{campaign_id}` | 5-15 минут | `campaign.updated` |
| Bid strategy | `bid_strategy:{campaign_id}` | 5-15 минут | `campaign.updated` |
| Budget availability | `budget:{advertiser_id}` | 10-60 секунд | `budget.changed`, `billing_event.processed` |
| Creative metadata | `creative:{creative_id}` | 15-60 минут | `creative.updated` |

TTL для бюджетов короче, чем для кампаний, так как учёт риска перерасходов важнее, чем
стоимость лишнего обновления.

## Прогрев кеша

Перед подключением новой DSP нужно прогреть кеш:

1. Загрузить активные кампании и таргетинг.
2. Загрузить доступный бюджет для активных рекламодателей.
3. Загрузить метаданные для кампаний в RTB.
4. Проверить Redis на тестовом потоке DSP.

## Инвалидация

Инвалидация на основе событий

- Campaign Service публикует `campaign.updated`, `campaign.paused`,
  `targeting.updated`;
- Finance Service публикует `budget.changed`;
- Bidding cache consumer обновляет или удаляет соответствующие ключи;
- Для страховки оставляем TTL на случай пропущенного события.
