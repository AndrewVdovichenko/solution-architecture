# Задание 2. Проектирование RTB и интеграции с DSP

В разделе описан целевой RTB-контур для подключения новой DSP-площадки:
выделенный Bidding Service, API Gateway, протоколы взаимодействия и паттерны
надежности.

## Состав артефактов

- [bidding-service.md](bidding-service.md) — спецификация сервиса ставок.
- [interaction.md](interaction.md) — схема взаимодействия с обоснованием выбора протоколов.
- [api-gateway.md](api-gateway.md) — дизайн API Gateway для DSP.
- [diagrams/rtb-sequence.puml](diagrams/rtb-sequence.puml) — диаграмма
  обработки bid request.
- [diagrams/rtb-component.puml](diagrams/rtb-component.puml) — диаграмма
  RTB-контура.

## Ключевое решение

RTB-запросы проходят через API Gateway в Bidding Service. В критическом пути
используются короткие синхронные вызовы и Redis-кеш. События ставок, кликов и
показов уходят в Kafka асинхронно, чтобы не увеличивать задержку ответа DSP.
