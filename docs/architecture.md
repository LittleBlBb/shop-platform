# Архитектура

## Сервисы

| Сервис | Порт (local) | БД | Доступ к данным | Роль |
|---|---|---|---|---|
| `api-gateway` | 8080 | — | — | Единая точка входа (Spring Cloud Gateway), маршрутизация, correlation-id |
| `order-service` | 8081 | `order_db` | Spring Data JPA | Заказы, оркестрация саги оформления |
| `inventory-service` | 8082 | `inventory_db` | JDBC (`JdbcTemplate`) | Остатки, резервирование товара |
| `payment-service` | 8083 | `payment_db` | Spring Data JPA | Имитация оплаты, идемпотентность платежей |
| `notification-service` | 8084 | `notification_db` | JDBC | Уведомления по событиям из Kafka |

Принципы:
- **Database per service** — у каждого сервиса своя БД, в чужую ходить нельзя. Локально и на стендах это отдельные базы в одном PostgreSQL.
- **Синхронно** (REST) — только запросы клиента через gateway и редкие запросы «нужно прямо сейчас».
- **Асинхронно** (Kafka) — всё, что меняет состояние другого сервиса. Надёжная публикация — через transactional outbox.
- Каждый сервис — независимый Maven-проект со своим wrapper, Dockerfile, Helm-values и CI-джобой (monorepo, но не монолит).

## Основной бизнес-поток (сага оформления заказа)

```
client → gateway → order-service: POST /orders           (status=NEW)
order-service      ──OrderCreated──────▶ inventory-service (резерв)
inventory-service  ──StockReserved / StockRejected──▶ order-service
order-service      ──PaymentRequested──▶ payment-service
payment-service    ──PaymentSucceeded / PaymentFailed──▶ order-service
order-service: CONFIRMED | CANCELLED (+ компенсация: StockReleased)
notification-service слушает итоговые события и «отправляет» уведомление
```

## Стенды

| Стенд | Где | Как попадает код |
|---|---|---|
| local | Windows-машина разработчика, `docker compose` | вручную |
| dev | k3s на сервере, namespace `shop-dev` | автоматически после merge в `main` |
| stage | k3s на сервере, namespace `shop-stage` | по тегу `vX.Y.Z` + ручное подтверждение (GitHub Environment) |

Общая инфраструктура на сервере: namespace `infra` (PostgreSQL, Kafka), `logging` (Elasticsearch, Logstash, Kibana).
Стенды разделены namespace'ами, отдельными базами (`order_db_dev` / `order_db_stage`) и префиксами топиков (`dev.` / `stage.`).

Доступ к кластеру и стендам — только через SSH-туннель (API и ingress слушают 127.0.0.1 на сервере):

```
ssh -L 6443:127.0.0.1:6443 -L 8081:127.0.0.1:8081 user@server
```
