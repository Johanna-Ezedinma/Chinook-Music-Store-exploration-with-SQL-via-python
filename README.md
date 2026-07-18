# Chinook Music Store: SQL Exploration (PostgreSQL)

> Exploration of the Chinook digital music store database in PostgreSQL, queried through Jupyter using `%%sql` magic.

**Author:** Johanna Ezedinma  
**Date:** July 2026

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/johanna-ezedinma/)
[![Medium](https://img.shields.io/badge/Medium-12100E?style=for-the-badge&logo=medium&logoColor=white)](https://medium.com/@johannaezedinma) [![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Johanna-Ezedinma)

---

## Project Summary

This project explores the Chinook digital music store database (11 tables: artists, albums, tracks, genres, media types, employees, customers, invoices, and invoice lines) entirely in PostgreSQL, queried through Jupyter using `%%sql` magic. It starts with schema discovery (table sizes, column data types, nullability), moves through core querying (filtering, sorting, grouping, aggregates), then into joins, subqueries, and window functions, and closes with business questions around sales performance and customer spend.

-# Chinook Music Store: SQL Exploration (PostgreSQL)

## Approach used, and why

SQL was chosen over pandas for this project because Chinook is a genuinely relational database, eight interconnected tables joined by real foreign keys, not a flat file. Answering questions like "which employee generated the most sales" or "which tracks rank highest by length within their own genre" requires joining across several tables and comparing individual rows against group-level values, exactly what SQL's `JOIN` and window function syntax is built to do directly, without first flattening everything into one table.

Running the queries through Jupyter (rather than a GUI client like pgAdmin) kept the SQL, its real output, and the reasoning behind each query together in one reviewable document, rather than split across a query tool and a separate write-up.

---

## Business problem solved

The project answers a set of operational questions a music store's management would realistically want answered: which sales agents are the top performers, which customers spend the most (and which spend above the store-wide average), which countries the customer base is concentrated in, which genres and media types are most popular by track count, and whether any staff currently outrank their own manager in the reporting hierarchy.

---

## SQL concepts used

- **Schema introspection** via `information_schema` (table sizes, column data types, nullability)
- **Filtering and sorting**: `SELECT`, `WHERE`, `ORDER BY`, `IN`
- **Aggregation**: `GROUP BY`, `HAVING`, `COUNT`, `SUM`, `AVG`
- **`COALESCE`** for readable handling of missing values at query time
- **`ALTER TABLE` / `UPDATE`** to derive a new column (`minutes`, computed from `milliseconds`)
- **Joins**: `INNER JOIN`, `LEFT JOIN`, and a self-join (an employee table joined to itself to compare each employee against their manager)
- **Subqueries**: a scalar subquery in a `WHERE` clause (looking up the Drama genre's ID), and a subquery used to compare each row against a dynamically computed average
- **Window functions**: `MAX() OVER (PARTITION BY ...)` and `DENSE_RANK() OVER (PARTITION BY ... ORDER BY ...)`
- **Common Table Expressions (`WITH`)**, used in the corrected manager-rank query below, to compute a derived value once and reuse it in a self-join

---

## How missing and duplicate data were handled

Chinook arrives as a pre-built, already-clean sample database. A full-row duplicate check was run across every table (customer, employee, invoice, invoice_line, track, album, artist, genre, media_type), confirming **zero exact duplicate rows anywhere**. Two things are worth separating out here, since they're easy to conflate with real data quality problems:

- **`company` and `state` are `NULL` for a meaningful share of customers.** These aren't errors, they reflect real individual customers (no company to list) and countries without a state/province system. Missing `company` values were surfaced directly with `IS NULL`; missing `state` values were handled at query time with `COALESCE(state, 'No State Provided')`, which improves readability in the output without altering the underlying data.
- **Repeated track names (e.g. "The Trooper" appearing 5 times) are not duplicate rows.** Checked directly against the data, each instance has a distinct `track_id`, a different `album_id`, and a slightly different runtime, confirming these are legitimate separate recordings (the same song appearing on different albums or compilations), not something to deduplicate.

---

## KPI to track

**Revenue per sales agent, tracked monthly**, would be the single most useful ongoing KPI for this business. It's already directly queryable from the existing schema (`employee` → `customer` → `invoice`), it reflects team performance in a way management can act on directly (coaching, staffing decisions, recognition), and trending it over time would reveal whether performance gaps between agents (Jane Peacock's $833.04 vs. Steve Johnson's $720.16, from the current snapshot) are consistent, structural patterns or just short-term fluctuation.

---

## Challenges faced

The trickiest part of this project wasn't SQL syntax, it was making sure a query's _logic_ actually matched its stated question, rather than just running without error and looking plausible.

The clearest example was the "which employees have a higher rank than their manager" query. The original version compared `employee_id` values and searched for a `'Director'` title as a stand-in for organizational rank. Neither holds up: `employee_id` is just insertion order, not rank, and Chinook has no `Director` title at all, confirmed directly against the data, which meant that half of the filter condition was always true. On top of that, the `WHERE` clause was missing parentheses, so `AND`/`OR` weren't grouping the way the query's own comment intended. It ran cleanly and returned a plausible-looking result (two employees), but traced back against the real hierarchy, it wasn't answering the question at all, it had quietly reduced down to "employees whose own title happens to contain 'Manager.'"

It was rebuilt to derive rank explicitly from job title (`General Manager`, then any other `Manager` title, then `Staff`/`Agent` titles) using a Common Table Expression, then compare each employee's rank directly against their manager's. Rerun against the live database, the corrected query returns **zero rows**, a legitimate finding (no rank inversions currently exist in this hierarchy), and a good reminder that a query returning _a_ result isn't the same as a query returning the _right_ one. It's worth manually tracing logic like this against a few real rows before trusting an output just because it looks reasonable.

---

## Tools

PostgreSQL, SQL (via `jupysql`/`ipython-sql` in Jupyter)

## Files

- `Chinook_Music_Store_exploration_with_SQL_via_python.ipynb`: full notebook, schema exploration, core and advanced queries, business questions

---

## 👤 Author

**Johanna Ezedinma**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/johanna-ezedinma/)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Johanna-Ezedinma)
[![Medium](https://img.shields.io/badge/Medium-12100E?style=for-the-badge&logo=medium&logoColor=white)](https://medium.com/@johannaezedinma)
