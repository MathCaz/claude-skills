---
name: sql-style-M
description: SQL style and structure rules for Claude-written SQL. Snowflake-tested but mostly vendor-neutral. Auto-applies whenever SQL is being written in dbt projects, .sql files, or Dagster asset files.
disable-model-invocation: false
paths: "**/*.sql,**/dbt_project.yml,**/dagster/**/*.py"
---

# SQL Style

Two sections: **General SQL rules** (any dialect) and **Snowflake-specific rules** (apply when the target is Snowflake). Some general rules are marked **(aspirational)** — they pull new code toward the standard; legacy code is what it is.

Real-world reference: a well-sectioned feature-engineering model (numbered sections, source CTEs, progressive JOIN CTEs) is the canonical example of these conventions in action — especially the sectioning and progressive-JOIN patterns below.

---

## Section 1 — General SQL rules

### Layout & syntax

- **Keywords UPPERCASE** — `SELECT`, `FROM`, `WHERE`, `INNER JOIN`, `QUALIFY`, etc. (needed for SQL syntax highlighting inside Python string literals).
- **Identifiers lower_snake_case** — `user_id`, `order_date`. No `.` in names. No leading digits.
- **Trailing commas, not leading.**
- **4-space indent** inside SELECT lists, CTE bodies, subqueries, and `ON` continuation lines.
- **Spaces around operators** — `price > 100`, `a.id = b.user_id`.
- **Parentheses for complex logic** — group conditions in WHERE / CASE for clarity.
- **Single quotes for string literals** — `'active'`, `'2026-01-01'`.

### Aliases

- **Always `AS`** for aliases — `users AS usr`, `total AS order_total` *(aspirational)*.
- **Descriptive initials** — `usr`, `ord`, `prd`, `evt` — **never single letters** (`u`, `o`, `a`).
- **Cap at 3 letters**; extend to 4 only when 3 would collide (e.g., `payments` vs `payouts` → `pmt` / `pyt`).
- **Same length per CTE/block** — vertically aligned `AS` columns *(aspirational)*.
- **Same alias for the same table** across the entire query, unless impossible (self-joins).
- **No alias needed in single-table CTEs** — no JOIN, no ambiguity, no alias.

```sql
-- Good
SELECT
    usr.email,
    ord.total
FROM users  AS usr
JOIN orders AS ord
    ON usr.id = ord.user_id
```

### Clauses

- **Clause order is a hard rule:**
  `SELECT → FROM → JOIN → WHERE → GROUP BY → HAVING → QUALIFY → ORDER BY → LIMIT`.
- **SELECT layout: stacked.** SELECT keyword at column 0, each column on its own line indented 4 spaces, trailing commas.

```sql
SELECT
    col_a,
    col_b,
    col_c
FROM my_table
```

- **Column grouping inside wide SELECTs** — when a SELECT has **10+ columns**, group them logically with `-- group title` comment lines:

```sql
SELECT
    -- Identifiers
    event_id,
    external_job_id,

    -- Event attributes
    event_code,
    event_reason,
    event_type,

    -- Time features
    DATE_TRUNC('hour', event_datetime) AS rounded_event_ts,
    DAYOFWEEKISO(event_datetime)       AS weekday_iso
```

### CTEs

- **No CTEs for very simple queries.** If the query is a single-table SELECT with simple WHERE filtering, no joins, no aggregations, no window functions, no subqueries — write it directly without CTEs. The rest of the CTE rules below apply *only when the query actually needs CTEs*.
- **CTE layout:** name at column 0, body indented 4 spaces, blank line between CTEs.

```sql
WITH

active_users_cte AS (
    SELECT
        user_id,
        email
    FROM users
    WHERE is_active = TRUE
),

recent_orders_cte AS (
    SELECT
        order_id,
        user_id
    FROM orders
    WHERE order_date >= '2026-01-01'
)
```

- **CTE naming:** use the `_cte` suffix — `active_users_cte`, `simple_features_cte`, etc.
- **Source CTE pattern.** Unless the query is very simple, the **top of the query** has dedicated "source CTEs" that:
  - read from exactly **one** upstream table/asset (`{{ ref(...) }}` or `{{ source(...) }}` in dbt)
  - **list all columns explicitly** (no `SELECT *` here)
  - may filter rows as needed
  - serve as the only source of truth — all downstream CTEs reference these, not the raw refs
- **`SELECT *`:** OK in **middle CTEs** (e.g., passing through after a JOIN). **Never** in source CTEs. **Never** in the final SELECT.

#### Sectioning

For non-trivial queries, organize CTEs into numbered sections. Match this convention:

- A **`---`...`---` dash separator** (~126 dashes) above every section / sub-section
- A **`-- N️⃣ Section title`** line — one line preferred; multi-line `--` description allowed when needed
- **Sub-numbers `N️⃣.M️⃣`** when a section groups multiple related CTEs
- Use **`🔟`** for top-level section 10; use `1️⃣0️⃣` inside sub-numbers; `1️⃣1️⃣` for 11, etc.
- Each CTE's purpose is captured by its section/sub-section header (no extra `--` line needed per CTE — the header serves that role)

```sql
------------------------------------------------------------------------------------------------------------------------------
-- 1️⃣ Load and filter source tables
events_cte AS (...),
users_cte AS (...),

------------------------------------------------------------------------------------------------------------------------------
-- 5️⃣ Encoding for embedding: Map categorical features to embedding index IDs

------------------------------------------------------------------------------------------------------------------------------
-- 5️⃣.1️⃣ Join user segment embedding
with_user_segment_embedding_cte AS (...),

------------------------------------------------------------------------------------------------------------------------------
-- 5️⃣.2️⃣ Join product category embedding
with_product_category_embedding_cte AS (...),
```

### Subqueries

- **Prefer CTEs over subqueries by default.** A subquery may remain inline only if it's clearly simple (e.g., a scalar `(SELECT MAX(...))` used in one place).
- **Hard cap: never more than 1 level of nesting** — no subquery inside a subquery, ever.

### JOINs

- **Always explicit JOIN type** — `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `FULL JOIN`. **Never** bare `JOIN`.
- **Never implicit comma joins** (no `FROM a, b WHERE a.id = b.id`).
- **One JOIN per line.** `ON` clause on the **next line, indented 4**. Multi-condition `ON` continues with `AND` on indented lines.
- **`ON` alias order follows FROM/JOIN reading order** — the alias that appeared first goes on the left:

```sql
FROM users         AS usr
INNER JOIN orders  AS ord
    ON usr.id = ord.user_id           -- usr first → usr on left
LEFT JOIN items    AS itm
    ON ord.id = itm.order_id          -- ord came before itm → ord on left
    AND itm.is_active = TRUE
```

- **JOIN placement:**
  - **Default:** do JOINs in the **final SELECT** (or in the smallest possible set of CTEs).
  - **Exception:** progressive JOIN CTEs are allowed for **feature engineering / ML pipelines** when each enrichment step is a meaningfully named, individually inspectable transformation (one CTE = one JOIN). Use judgment.

### WHERE patterns (neutral-element prefix)

For a **single static condition**, just write the condition directly — no neutral element needed:

```sql
WHERE _snowflake_deleted = FALSE
```

For **multiple conditions** OR **any Jinja-templated condition**, prefix with the appropriate neutral element so every condition line is uniform and the first condition can be conditionally absent:

- **`WHERE 1 = 1`** for AND-chained filters (the common case)
- **`WHERE 0 = 1`** for OR-chained filters (less common; use when it improves readability)

```sql
-- AND chain
FROM events
WHERE 1 = 1
    AND event_type = 'ASSIGNMENT'
    {% if is_incremental() %}
        AND ingested_at >= DATEADD(DAY, -7, CURRENT_TIMESTAMP)
    {% endif %}

-- OR chain
FROM events
WHERE 0 = 1
    {% if include_active %}
        OR status = 'active'
    {% endif %}
    {% if include_pending %}
        OR status = 'pending'
    {% endif %}
```

### Audit columns

- **Every asset must have a `created_at` column** — non-negotiable. Every model/table carries a row-creation timestamp.
- **When `created_at` is propagated from an upstream model, suffix with the source model name** — `created_at__stg_events`, `created_at__int_orders`. Makes lineage visible in the row itself and avoids collisions when a downstream model pulls `created_at` from multiple sources.
- The **freshly-generated `created_at` in the current model stays unsuffixed** — e.g. `CURRENT_TIMESTAMP AS created_at`. So a downstream model can pull `created_at` from an upstream and end up with both `created_at` (its own) and `created_at__<upstream>` (inherited).

```sql
SELECT
    -- Identifiers
    event_id,

    -- Inherited lineage
    stg.created_at   AS created_at__stg_events,
    dim.created_at   AS created_at__dim_customers,

    -- This model's own timestamp
    CURRENT_TIMESTAMP AS created_at
FROM {{ ref('stg_events') }} AS stg
LEFT JOIN {{ ref('dim_customers') }} AS dim
    ON stg.customer_id = dim.customer_id
```

### Comments

- Comment **non-obvious logic**, **big steps** (CTE sections already covered above), and **non-trivial parts**. The goal: someone reading the file should grasp it fast without digging into every line.
- Don't comment what good naming already conveys.

---

## Section 2 — Snowflake-specific rules

Apply when the target is Snowflake. Code examples: [references/snowflake.md](references/snowflake.md).

1. `QUALIFY` to filter window functions — no subquery wrap.
2. Reuse computed column aliases in the same SELECT (important).
3. Trailing comma after the last column is allowed and encouraged (cleaner diffs).
4. `IFF(cond, x, y)` over `CASE WHEN` for simple binary cases.
5. `* EXCLUDE (col1, col2)` instead of listing 30 columns to drop 2.
6. Snowflake datetime functions: `DATE_TRUNC`, `DATEDIFF`, `DATEADD`, `EXTRACT`, `DAYOFWEEKISO`, `DATE_PART`.
7. Snowflake array functions: `ARRAY_CONSTRUCT`, `ARRAY_SIZE`, `ARRAY_EXCEPT`, `ARRAY_CONTAINS`, `ARRAY_AGG`.
8. `HASH()` for deterministic 64-bit surrogate keys / encoded IDs.

---

## Section 3 — Reference example

The strongest demonstration of these conventions applied together is a **feature-engineering model for an ML pipeline** — the shape that exercises every rule at once: source CTEs at the top, numbered sections with sub-numbers, progressive JOIN CTEs (one enrichment per CTE), column grouping in a wide final `SELECT`, `QUALIFY` for last-event-per-key selection, and Snowflake functions (`HASH`, `DATE_TRUNC`, `IFF`). When you write one, build it to this standard and it becomes the reference for the rest of the codebase.
