---
title: "Writing SQL in Big Data System"
datePublished: Wed Apr 16 2025 04:04:34 GMT+0000 (Coordinated Universal Time)
cuid: cm9jeqtr6000j0ajs83wu4ni2
slug: writing-sql-in-big-data-system
tags: big-data, sql

---





Working with massive datasets changes how you approach writing SQL. It’s not just about writing correct queries — it’s about writing *smart* ones that respect scale, cost, and the distributed nature of the underlying systems. Here are some hard-earned lessons and best practices from my journey wrangling SQL in big data environments like Hive, Presto, BigQuery, and Spark SQL.

---

## 1. **Push Work Down — Let the Engine Work for You**

In distributed systems, each stage of the query execution plan (especially filters and projections) matters. The golden rule:

> **Always filter early, project only what you need.**

By reducing the amount of data read and shuffled, you cut down IO, memory usage, and execution time significantly. For example:

```sql
-- Avoid
SELECT * FROM events WHERE event_type = 'purchase';

-- Prefer
SELECT user_id, event_time FROM events WHERE event_type = 'purchase';
```

---

## 2. **Understand Partitioning and Bucketing**

Big data systems usually partition tables to improve query performance. If you ignore partitioning, you're likely scanning terabytes unnecessarily.

> Know your table’s partition columns — and **use them in WHERE clauses**.

```sql
-- Leverage partition pruning
SELECT COUNT(*) 
FROM logs 
WHERE event_date = '2025-04-15';
```

If you're designing tables, partition on columns with high cardinality and frequent filtering. Avoid over-partitioning (e.g., hourly), which leads to small files and metadata overhead.

---

## 3. **Watch for Data Skew**

One sneaky performance killer is skewed data — where a few values dominate a join or group by. This causes uneven workload distribution, leaving some workers idle while others are overloaded.

**Telltale signs:**
- Stages stuck at 99% for a long time.
- Huge temp spill files on specific workers.

### How to mitigate:
- **Salting keys** (add a random suffix to break hotspots).
- Use approximate aggregations where precision isn't mission-critical.
- Rewrite joins as map-side when appropriate.

---

## 4. **Avoid Cross Joins and Cartesian Products**

It seems obvious, but in massive datasets, even an accidental cross join can generate *petabytes* of intermediate data. Be explicit in join conditions and cautious with one-to-many relationships.

```sql
-- Danger zone
SELECT a.*, b.* 
FROM users a, events b 
WHERE a.user_id = b.user_id;  -- missing JOIN syntax is risky

-- Safer
SELECT a.*, b.*
FROM users a
JOIN events b ON a.user_id = b.user_id;
```

---

## 5. **Use WITH Clauses and CTEs Judiciously**

CTEs (Common Table Expressions) improve readability, but **some engines materialize them**, which can result in recomputation. For frequently reused logic or expensive computations, **cache results or persist to a temp table**.

```sql
-- Avoid repeating subqueries in massive datasets
WITH filtered AS (
  SELECT user_id, SUM(amount) AS total_spent
  FROM transactions
  WHERE event_date >= '2025-01-01'
  GROUP BY user_id
)
SELECT * FROM filtered WHERE total_spent > 1000;
```

---

## 6. **Understand Cost Models and Query Plans**

A habit worth cultivating: **always inspect the execution plan**. Use `EXPLAIN` or visual tools to:
- Spot full scans
- Understand join order
- Check shuffle/sort stages

This is especially critical in systems like Spark or Presto where optimizers can make suboptimal decisions.


> 💡 Pro Tip: 
>
>In BigQuery, filter on partitioned and clustered columns to benefit from partition pruning and block elimination.


---

## 7. **Beware of Wide Joins and Explosions**

Be mindful of fan-out joins, especially when joining arrays or struct types that can cause row multiplication.

> If you’re joining nested fields, **flatten responsibly**.

In systems like Spark SQL:
- Use `explode()` with care.
- Always check the row count before and after to detect accidental data amplification.

---

## 8. **Monitor Query Cost and Audit Usage**

In systems like BigQuery or Athena, you pay by data scanned. Even a simple `SELECT COUNT(*) FROM table` can cost dollars if you don't project partitions or filter early.

Set up dashboards:
- Query cost by user or team
- Top tables by scan volume
- Longest running queries

Cost-awareness = performance discipline.

---

## 9. **Materialize Strategic Results**

Instead of chaining many heavy transformations, **break your pipeline**:
- Materialize intermediate results (to a temp or summary table).
- Validate, then continue downstream.

This helps with reusability, debuggability, and disaster recovery.

---

## 10. **Automate Linting and Review of SQL**

Treat SQL as production code. Use tools that lint SQL for anti-patterns (e.g., full scans, cross joins) or enforce style consistency in your team.

For example:
- **SQLFluff** for syntax/style
- **Datafold or dbt** for CI/CD with data quality checks

---

# Final Thoughts

Working with SQL in big data is like driving a race car — you're dealing with speed, power, and risk. The key is to **design queries that respect scale**, understand the engine underneath, and develop a culture of experimentation and observability.

The faster you get at reading query plans, the faster you become at writing efficient SQL.

If you’re just starting out, pick one of these lessons and apply it to a real query — you’ll be surprised how far a small change can go at scale.
