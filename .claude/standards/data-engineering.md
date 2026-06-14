# Data Engineering Standard

> Mandatory standards for data pipelines, stream processing, and batch workloads. Enforced by `data-engineer` agent and CI gates.

---

## Core Principles

1. **Idempotency** — every pipeline step produces the same output given the same input, regardless of how many times it runs
2. **Observability** — every pipeline emits metrics, logs completion counts, and surfaces data quality violations
3. **Schema contracts** — all data crossing a pipeline boundary has a registered, versioned schema
4. **Failure isolation** — a single bad record never crashes a pipeline; dead-letter all unprocessable records

---

## Apache Kafka

### Producer Rules

- Set `acks=all` and `enable.idempotence=true` on every producer
- Never produce without a schema — register all schemas in the Schema Registry before first use
- Use the Outbox Pattern for transactional producers — write to outbox table in the same DB transaction, relay asynchronously
- Key messages by entity ID to guarantee partition affinity and ordering per entity

```python
# CORRECT — idempotent producer with schema
from confluent_kafka import Producer
from confluent_kafka.schema_registry import SchemaRegistryClient
from confluent_kafka.schema_registry.avro import AvroSerializer

producer = Producer({
    "bootstrap.servers": settings.kafka_brokers,
    "acks": "all",
    "enable.idempotence": True,
    "compression.type": "snappy",
})
```

### Consumer Rules

- Set `enable.auto.commit=false` — commit offsets only after successful processing
- Consume within a `try/except` and dead-letter unprocessable records — never skip silently
- Name consumer groups as `<service>-<topic>-consumer` (e.g., `order-service-payments-consumer`)
- Always define `auto.offset.reset=earliest` for new consumer groups on existing topics

```python
# CORRECT — manual commit, DLQ on error
for message in consumer:
    try:
        process(message)
        consumer.commit(message)
    except ProcessingError as exc:
        logger.error("DLQ: unprocessable message: key=%s", message.key(), exc_info=True)
        dlq_producer.produce(dlq_topic, key=message.key(), value=message.value())
        consumer.commit(message)
```

### Topic Naming

| Pattern | Format | Example |
|---|---|---|
| Domain event | `<domain>.<entity>.<event>` | `orders.order.placed` |
| Command | `<domain>.<entity>.commands` | `payments.payment.commands` |
| Dead letter | `<source-topic>.dlq` | `orders.order.placed.dlq` |
| Retry | `<source-topic>.retry.<n>` | `orders.order.placed.retry.1` |

---

## Apache Spark

### Job Structure

- One `SparkSession` per job — never create multiple sessions
- Read once, transform, write once — never re-read the same dataset mid-job
- Use DataFrame API (not RDD API) for all new code; use Spark SQL only for complex aggregations
- Partition output by date column: `df.write.partitionBy("year", "month", "day")`

```python
# CORRECT — structured Spark job
from pyspark.sql import SparkSession
from pyspark.sql import functions as F

def run(spark: SparkSession, input_path: str, output_path: str) -> None:
    df = spark.read.parquet(input_path)
    result = (
        df.filter(F.col("status") == "COMPLETED")
          .groupBy("customer_id")
          .agg(F.sum("amount").alias("total_spend"))
    )
    result.write.mode("overwrite").partitionBy("year", "month").parquet(output_path)

if __name__ == "__main__":
    spark = SparkSession.builder.appName("CustomerSpendAggregation").getOrCreate()
    run(spark, sys.argv[1], sys.argv[2])
```

### Quality Checks

- Assert row counts before and after transformations for critical pipelines
- Use `pydeequ` or `great_expectations` for schema and statistical data quality checks
- Fail the job if data quality checks fail — never silently continue with bad data

---

## dbt

### Model Conventions

- Staging models (`stg_*`): raw source renaming only — no business logic
- Intermediate models (`int_*`): joins, aggregations, business logic
- Mart models (`dim_*`, `fct_*`): final consumer-facing tables

```sql
-- CORRECT: staging model — rename only, no business logic
-- models/staging/stg_orders.sql
select
    id           as order_id,
    customer_id,
    created_at   as placed_at,
    status
from {{ source('raw', 'orders') }}
where _deleted_at is null

-- WRONG: business logic in staging
select *, total_amount * 1.2 as with_tax  -- business logic belongs in int_/fct_
from {{ source('raw', 'orders') }}
```

### Testing

- Every model has at minimum `not_null` and `unique` tests on its primary key
- Use `dbt-expectations` for statistical tests (value ranges, distributions)
- Freshness tests on all source tables using `freshness:` in `sources.yml`

### Materialisation Strategy

| Model Layer | Materialisation | Reason |
|---|---|---|
| Staging | `view` | Always fresh, no storage cost |
| Intermediate | `ephemeral` or `table` | Depends on reuse frequency |
| Mart | `table` or `incremental` | Optimised for consumer query performance |

---

## Idempotency Patterns

- **Batch jobs:** use `INSERT INTO ... ON CONFLICT DO NOTHING` or `MERGE` — never plain `INSERT`
- **Partitioned writes:** overwrite the entire partition — never append to existing partitions
- **Kafka consumers:** store processed event IDs in a deduplication table with TTL

```sql
-- CORRECT — idempotent upsert
INSERT INTO order_aggregates (order_id, total, updated_at)
VALUES (:order_id, :total, NOW())
ON CONFLICT (order_id) DO UPDATE
  SET total = EXCLUDED.total,
      updated_at = EXCLUDED.updated_at;
```

---

## Observability

- Emit pipeline metrics to CloudWatch / Prometheus: `records_read`, `records_written`, `records_failed`, `duration_seconds`
- Log job start and end with row counts at `INFO` level
- Alert on: job duration exceeding 2× historical p95, data quality check failures, DLQ depth > 0

---

## Anti-Patterns

| Anti-Pattern | Correct Alternative |
|---|---|
| `enable.auto.commit=true` in Kafka consumer | Manual commit after successful processing |
| Producing without a schema | Register Avro/Protobuf schema in Schema Registry |
| Business logic in staging dbt models | Move to intermediate/mart layer |
| Plain `INSERT` without deduplication | `UPSERT` / `MERGE` / `ON CONFLICT DO NOTHING` |
| Catching and ignoring bad records | Dead-letter queue with structured logging |
| RDD API in new Spark code | DataFrame/Dataset API |
| Hardcoded connection strings in job code | Environment variables / Secrets Manager |
| No data quality checks | `great_expectations` or `pydeequ` assertions |
