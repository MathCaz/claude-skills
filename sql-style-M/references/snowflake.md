# Snowflake-specific rules — full examples

Linked from `sql-style-M` SKILL.md, Section 2.

These apply when the target dialect is Snowflake. Skip when writing for Postgres / BigQuery / etc.

1. **`QUALIFY` for filtering window functions** — no subquery wrap:

   ```sql
   -- Good
   SELECT *
   FROM events
   QUALIFY ROW_NUMBER() OVER (
       PARTITION BY event_id
       ORDER BY created_at DESC
   ) = 1

   -- Bad
   SELECT * FROM (
       SELECT *, ROW_NUMBER() OVER (...) AS rn
       FROM events
   )
   WHERE rn = 1
   ```

2. **Reuse computed column aliases in the same SELECT** (important):

   ```sql
   SELECT
       a + b           AS sum_ab,
       sum_ab * 2      AS doubled,        -- reuses sum_ab in the same SELECT
       sum_ab / total  AS ratio           -- and again
   FROM t
   ```

3. **Trailing comma on the last column is allowed and encouraged** — produces cleaner diffs when adding/removing columns:

   ```sql
   SELECT
       a,
       b,
       c,        -- trailing comma OK in Snowflake
   FROM t
   ```

4. **Prefer `IFF(cond, x, y)` over `CASE WHEN`** for simple binary cases:

   ```sql
   IFF(qty > 0, 'positive', 'zero_or_neg')
   ```

5. **`* EXCLUDE (col1, col2)`** when selecting "everything except a few" — avoids listing 30+ columns to drop 2:

   ```sql
   SELECT * EXCLUDE (id, created_at) FROM t
   ```

6. **Prefer Snowflake datetime functions:** `DATE_TRUNC`, `DATEDIFF`, `DATEADD`, `EXTRACT`, `DAYOFWEEKISO`, `DATE_PART`.

7. **Prefer Snowflake array functions:** `ARRAY_CONSTRUCT`, `ARRAY_SIZE`, `ARRAY_EXCEPT`, `ARRAY_CONTAINS`, `ARRAY_AGG`.

8. **`HASH()` for deterministic surrogate keys / encoded IDs** — fast, deterministic, 64-bit:

   ```sql
   HASH(order_id) AS order_id_int
   ```
