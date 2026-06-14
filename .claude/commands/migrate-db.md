# /migrate-db — Database Migration Generator

Generate a safe, production-ready Flyway or Liquibase migration with rollback script and risk assessment.

## Usage

```
/migrate-db "Add email_verified column to customers table (nullable boolean, default false)"
/migrate-db "Create orders_archive table and move orders older than 2 years"
/migrate-db "Add index on orders.customer_id and orders.created_at"
```

## What This Command Does

1. Activates the `dba-advisor` agent
2. Assesses the migration risk level: **Low / Medium / High**
3. Generates the forward migration script following project naming conventions
4. Generates the rollback script (or explains why the migration is forward-only)
5. Flags any online DDL requirements (large table DDL, index CONCURRENTLY)
6. Estimates execution time and whether a maintenance window is required

## Risk Levels

| Level | Examples | Deployment Approach |
|---|---|---|
| **Low** | Add nullable column, create index CONCURRENTLY, add view | Deploy with application release |
| **Medium** | Add NOT NULL column with default, add constraint, rename with dual-write | Deploy before app release in separate step |
| **High** | Large data migration (>100k rows), table rebuild, remove column still referenced | Maintenance window + DBA review |

## Output Format

### Migration Assessment
- Risk level with justification
- Estimated rows affected
- Lock type and duration
- Whether online DDL is required

### Forward Migration (Flyway)
```sql
-- V<version>__<description>.sql
-- Risk: Low | Estimated duration: <1s | Lock: None (CONCURRENTLY)
-- Rollback: V<version+1>__rollback_<description>.sql

-- Add nullable column — no lock, safe to run with traffic
ALTER TABLE customers ADD COLUMN email_verified BOOLEAN DEFAULT FALSE;

-- Create index without locking table
CREATE INDEX CONCURRENTLY idx_customers_email_verified
    ON customers (email_verified)
    WHERE email_verified = TRUE;
```

### Rollback Script
```sql
-- V<version+1>__rollback_<description>.sql  (or manual rollback steps)
DROP INDEX CONCURRENTLY IF EXISTS idx_customers_email_verified;
ALTER TABLE customers DROP COLUMN IF EXISTS email_verified;
```

### Pre-Deployment Checklist
- [ ] Migration tested against a production data volume clone
- [ ] Rollback script tested
- [ ] Application code handles both old and new schema (if deploying simultaneously)
- [ ] DBA reviewed if risk is Medium or High

## Tips

- Always specify the table size when known (row count) — it changes the risk assessment
- For large data migrations, request a separate `/migrate-db` for the backfill job
- Index creation on production tables should always use `CONCURRENTLY`
- Never add `NOT NULL` to an existing column without first ensuring all rows have a value
