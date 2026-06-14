# Event-Driven Architecture Standard

> Mandatory standards for asynchronous, event-driven systems. Complements `integration-standard.md`. Enforced by `architect` and `ci-engineer` agents.

---

## Foundational Rules

1. **Events describe facts** — past tense, immutable: `OrderPlaced`, `PaymentAuthorised`, not `PlaceOrder`
2. **Consumers are idempotent** — the same event delivered twice must produce the same result as delivered once
3. **Producers do not know their consumers** — no shared code between producer and consumer domain logic
4. **Schema contracts are versioned** — breaking changes require a new event type, not modification of an existing one
5. **Every dead-letter queue is monitored** — DLQ depth > 0 triggers an alert

---

## Event Schema

All events must include this envelope:

```json
{
  "eventId": "550e8400-e29b-41d4-a716-446655440000",
  "eventType": "orders.order.placed",
  "schemaVersion": "1.0",
  "timestamp": "2024-01-15T10:30:00.000Z",
  "correlationId": "req-abc-123",
  "producerService": "order-service",
  "payload": {
    "orderId": "ord-789",
    "customerId": "cust-456",
    "totalAmount": "149.99"
  }
}
```

**Rules:**
- `eventId`: UUID v4, unique per event instance — used for deduplication
- `timestamp`: ISO-8601 UTC — never local time
- `correlationId`: propagate from the originating HTTP request or parent event
- `payload`: domain data only — no infrastructure metadata in payload

---

## Event Naming

| Pattern | Format | Example |
|---|---|---|
| Domain event | `<domain>.<entity>.<past-tense-verb>` | `orders.order.placed` |
| Integration event (crossing context boundary) | `<source-domain>.integration.<entity>.<verb>` | `orders.integration.order.confirmed` |
| Dead letter | `<source-topic>.dlq` | `orders.order.placed.dlq` |
| Retry | `<source-topic>.retry.<attempt>` | `orders.order.placed.retry.1` |

---

## Schema Evolution Rules

| Change Type | Allowed? | Strategy |
|---|---|---|
| Add optional field to payload | Yes | Consumers must tolerate unknown fields (use lenient deserialisation) |
| Remove a field | No | Deprecate field (set to null); only remove after all consumers migrated |
| Change field type | No | Add a new field with new type; deprecate old field |
| Rename a field | No | Add new field, deprecate old, remove after migration |
| Breaking change | No | Create a new event type (`orders.order.placed.v2`); run in parallel during migration |

Consumers must use `FAIL_ON_UNKNOWN_PROPERTIES = false` (Jackson) or equivalent — they must not crash when the producer adds new fields.

---

## Outbox Pattern

Never publish events directly from within a database transaction. Use the Outbox Pattern to guarantee at-least-once delivery without distributed transactions.

```
Transaction:
  1. Write domain state change to domain tables
  2. Write event to outbox table (same transaction)

Relay process (separate, scheduled):
  3. Read unprocessed outbox rows
  4. Publish to message broker
  5. Mark outbox row as processed
```

```java
// CORRECT — event written to outbox in same transaction as domain change
@Transactional
public Order placeOrder(PlaceOrderCommand cmd) {
    Order order = Order.place(cmd);
    orderRepository.save(order);
    outboxRepository.save(OutboxEvent.from(new OrderPlacedEvent(order)));  // same TX
    return order;
}

// WRONG — publish in catch block or after commit (loses events on crash)
public Order placeOrder(PlaceOrderCommand cmd) {
    Order order = Order.place(cmd);
    orderRepository.save(order);
    eventBus.publish(new OrderPlacedEvent(order));  // not transactional
    return order;
}
```

---

## Idempotent Consumers

Every consumer must handle duplicate delivery without side effects.

**Strategy 1 — Deduplication table:**

```sql
CREATE TABLE processed_events (
    event_id UUID PRIMARY KEY,
    processed_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- In consumer: check before processing
INSERT INTO processed_events (event_id) VALUES (:eventId)
ON CONFLICT (event_id) DO NOTHING;
-- If 0 rows inserted → duplicate → skip processing
```

**Strategy 2 — Natural idempotency:**

Design the state change to be naturally idempotent — e.g., `UPDATE orders SET status = 'CONFIRMED' WHERE id = ? AND status != 'CONFIRMED'`. Re-processing has no effect.

---

## Dead Letter Queue

Every consumer must have a DLQ. Structure:

- DLQ name: `<source-queue-name>-dlq`
- Message in DLQ retains original message body + failure metadata header
- Alarm: `DLQMessageCount > 0` for any sustained period
- Runbook: documented process for replaying DLQ messages after fixing the root cause

```yaml
# AWS CDK — SQS with DLQ
const dlq = new sqs.Queue(this, 'OrderPlacedDlq', {
  queueName: 'orders-order-placed-dlq',
  retentionPeriod: Duration.days(14),
});

const queue = new sqs.Queue(this, 'OrderPlacedQueue', {
  queueName: 'orders-order-placed',
  deadLetterQueue: { queue: dlq, maxReceiveCount: 3 },
  visibilityTimeout: Duration.seconds(30),
});
```

---

## Retry Policy

| Scenario | Strategy |
|---|---|
| Transient infrastructure failure (DB unavailable) | Exponential backoff with jitter; max 3 attempts before DLQ |
| Poison pill (message always fails) | DLQ immediately after max attempts; alert |
| Dependency temporarily unavailable | Retry with circuit breaker open; messages stay in-flight |

Backoff formula: `wait = min(base × 2^attempt + jitter, max_wait)`
- `base` = 1s, `max_wait` = 30s, `jitter` = random(0, base)

---

## Circuit Breaker (Resilience4j)

Configure circuit breakers on all synchronous calls made from event consumers to downstream services.

```yaml
resilience4j:
  circuitbreaker:
    instances:
      inventory-service:
        failure-rate-threshold: 50          # open circuit at 50% failure rate
        slow-call-rate-threshold: 80        # also open if 80% of calls are slow
        slow-call-duration-threshold: 2000ms
        wait-duration-in-open-state: 30s    # time before attempting half-open
        permitted-number-of-calls-in-half-open-state: 5
        sliding-window-size: 20
        minimum-number-of-calls: 10
```

---

## Event Catalog

All events produced by a service must be registered in the project Event Catalog at `docs/events/catalog.md`:

| Event Type | Producer | Consumers | Schema Version | Retention |
|---|---|---|---|---|
| `orders.order.placed` | order-service | payment-service, inventory-service | 1.2 | 7 days |

---

## Anti-Patterns

| Anti-Pattern | Correct Alternative |
|---|---|
| Publish events inside a catch block | Outbox Pattern — publish in same transaction as domain change |
| Non-idempotent consumers | Deduplication table or natural idempotency |
| No DLQ | Every queue has a DLQ with a monitored alarm |
| Sharing consumer code with producer | Event contract via schema only |
| Mutable events | Events are immutable facts — never update a published event |
| Including PII in event headers | PII in encrypted payload only; never in metadata |
| Schema change without versioning | Add fields (backward compatible) or create new event type |
| Synchronous chain replacing async | Max 3-hop sync chains; beyond that, use events |
