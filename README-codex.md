# README-codex — Metrics Monitoring & Alerting (Spring Boot, Java, Gradle)

## Цель
Сгенерировать систему мониторинга:
- collector (pull + push ingest)
- ingestion-service (валидирует и пишет в TS storage)
- query-service (API запросов)
- alerting-service (оценка правил + дедуп + routing)
- notification-gateway (mock)
- contracts + infra

## Стек
- Java 17+, Spring Boot 3.x, Gradle
- Storage: ClickHouse (таблица time-series) или Timescale/Postgres
- Cache: Redis (optional)
- Queue: Kafka optional (для decouple)
- Actuator + Micrometer

## Модули
/modules
  /contracts
  /collector
  /ingestion-service
  /query-service
  /alerting-service
  /notification-gateway
/infra/docker-compose.yml
/docs (C4)

## Данные
MetricPoint:
- metric: String
- labels: Map<String,String>
- ts: Instant
- value: double

Хранилище (пример ClickHouse):
- metric String
- labels Map(String,String) или отдельные колонки (для упрощения)
- ts DateTime
- value Float64

Важно: контролировать cardinality (labels explosion) — добавить лимиты/валидацию.

## API
### collector
- Pull: `/scrape` вызывает targets (mock registry)
- Push: `POST /v1/ingest`

### query-service
- `GET /v1/query?metric=...&from=...&to=...&labels=...&agg=...`
- `GET /v1/series?metric=...` (metadata optional)

### alerting-service
- CRUD правил: `POST /v1/rules`, `GET /v1/rules`
- Runtime: scheduler каждые N секунд вызывает query-service

### notification-gateway
- `POST /v1/notify` принимает алёрты, логирует/сохраняет

## Порядок генерации
1) Gradle multi-module, shared libs (spring-boot-starter-web, validation, actuator).
2) contracts: DTO + OpenAPI fragments.
3) infra: clickhouse + redis + (optional kafka).
4) ingestion-service:
   - принимает MetricPoint (direct or from kafka)
   - batch inserts в storage
   - integration tests (Testcontainers)
5) query-service:
   - SQL генератор агрегаций (avg/sum/p95)
   - cache layer (optional) через Redis
6) alerting-service:
   - Rule model: name, query, threshold, forDuration, labels, annotations, route
   - Scheduler (Spring @Scheduled)
   - Dedup store (in-memory + TTL, или Redis)
   - Routing policy (простая: по label team/severity)
   - Notification retries
7) collector:
   - pull targets из списка (application.yml)
   - преобразует scraped формат в MetricPoint
8) e2e:
   - ingest -> query -> rule fires -> notification received

## Done criteria
- `docker compose up`
- push ingest работает
- query возвращает time-series
- alerting триггерит и отправляет в notification-gateway
