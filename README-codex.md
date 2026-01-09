# README-codex — Metrics Monitoring and Alerting

## Цель
Сгенерировать минимально рабочую систему:
- collector (pull + push endpoint)
- ingestion + TSDB write
- query-service
- alerting-engine
- notification-gateway (mock)
- web UI (опционально: простой dashboard)

## Рекомендуемый стек (для скорости)
- TSDB: ClickHouse (таблица time-series) или InfluxDB
- Queue: Redpanda/Kafka (опционально)
- Cache: Redis
- Alert delivery: mock HTTP receiver + логирование

## Контракты
### Ingest API
- `POST /ingest` body: {metric, labels, ts, value}
### Query API
- `GET /query?metric=...&from=...&to=...&labels=...&agg=...`
### Alerts
- Rule YAML: name, expr, threshold, for, labels, annotations, route

## Порядок генерации
1) Infra: docker-compose (tsdb, redis, kafka optional).
2) Collector:
   - pull targets через service discovery (mock JSON)
   - push endpoint
3) Ingestion:
   - validate
   - write to TSDB
4) Query service:
   - basic query + aggregation + caching
5) Alerting:
   - scheduler
   - evaluate rules via query service
   - dedup + routing
   - send to notification gateway
6) Тесты:
   - e2e: ingest -> query -> alert fired -> notification received
