# Масштабирование баз данных

## Репликация

| Сервис | Стратегия |
| --- | --- |
| Campaign | Паттерн read-replica для дашбордов |
| Finance | Паттерн multi-master |
| Bidding | Паттерн master-slave |
| Analytics | Паттерн master-slave |
| Statistics | Паттерн read-replica |

## Шардирование

Шардирование вводится в том случае, когда репликации и/или оптимизации запросов
не достаточно.

| Данные | Ключ шардирования | Причина |
| --- | --- | --- |
| Campaign analytics | `advertiser_id` / `campaign_id` | Изоляция крупных рекламодателей |
| Bid events | `partner_id` + time bucket | Удобно анализировать SLA и нагрузку DSP |
