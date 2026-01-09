# Metrics Monitoring and Alerting System

Система метрик уровня инфраструктуры и приложений: сбор, транспорт, хранение (time-series), запросы для дашбордов и алёртов, доставка уведомлений.

## Компоненты (концептуально)
1) Data collection (pull/push)
2) Data transmission (pipeline/queue)
3) Storage (TSDB)
4) Query service (агрегации/запросы)
5) Alerting (правила, дедуп, routing)
6) Visualization (UI)

## Требования
### Functional
- Приём метрик (CPU, memory, RPS, latency, queue depth и т.п.)
- Labels/tags (host, region, service, env, ...)
- Retention policy (пример: 7d raw, 30d 1m, 1y 1h downsample)
- Запросы для графиков + алёрт-правила
- Доставка алёртов (email, sms, pager, webhooks)

### Non-functional
- Высокая доступность и надёжность (critical alerts)
- Низкая задержка запросов для UI/алёртов
- Высокая write-нагрузка, спайки read-нагрузки
- Гибкость расширения пайплайна

## Архитектура
### Collection: Pull vs Push
- Pull: проще health-check, удобнее дебаг
- Push: проще за NAT/firewall, подходит для short-lived jobs через gateways

### Pipeline
- Коллекторы -> (Kafka/queue как буфер) -> ingestion/consumers -> TSDB
- Cache layer для ускорения запросов
- Query service как слой абстракции над TSDB (опционально)

## Модель данных
- metric_name
- labels (k=v)
- datapoints: (timestamp, value)

## Хранение
- TSDB (InfluxDB/Prometheus remote storage/ClickHouse/Timescale)
- Сжатие и кодирование, downsampling, rollup

## Alerting
- Rule evaluation (например, каждые 10s/30s)
- Дедуп/группировка, silencing, routing по политикам
- Надёжная доставка (retries, escalation)

## Масштабирование
- Шардирование по metric_name/tenant
- Kafka partitions по metric_name или hash(labels)
- Горизонтальный ingestion
- Query cache + pre-aggregation

## Надёжность
- Очередь как защита от недоступности TSDB
- Репликация TSDB/кластер
- Backpressure на ingestion

## Observability
- Ingestion lag, write failures, query latency p95/p99
- Cardinality explosion (число уникальных series)
- Alert delivery latency, dropped notifications
