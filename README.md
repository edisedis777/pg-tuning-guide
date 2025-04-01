# PostgreSQL Tuning Guide
[![Visual Studio Code](https://custom-icon-badges.demolab.com/badge/Visual%20Studio%20Code-0078d7.svg?logo=vsc&logoColor=white)](#)
[![Markdown](https://img.shields.io/badge/Markdown-%23000000.svg?logo=markdown&logoColor=white)](#)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)


A practical guide for tuning PostgreSQL parameters based on workload and hardware specifications. 
This project provides example SQL scripts and recommendations for tuning key PostgreSQL parameters, including memory, parallelism, JIT, and connection settings. 

---

## Project Overview

This file serves as a resource for tuning PostgreSQL parameters.

### Disclaimer
The recommended values serve as a starting point. Optimal settings depend on workload type, hardware specifications, and database size. 
Always analyze your system’s performance and adjust based on real-world usage and monitoring.

---

## Project Structure (Conceptual)

While everything is in this README, here’s how the content is logically organized:
- **Memory-Related Parameters**: Tuning `work_mem`, `shared_buffers`, etc.
- **Parallelism-Related Parameters**: Optimizing `max_parallel_workers`, etc.
- **JIT-Related Parameters**: Configuring `jit`, `jit_above_cost`, etc.
- **Connection-Related Parameters**: Adjusting `max_connections`, etc.
- **Helper Scripts**: Tools for applying settings and monitoring.

---

## Setup Instructions

1. Ensure PostgreSQL is installed and running on your system.
2. Copy and paste the SQL scripts below into your PostgreSQL client (e.g., `psql -U postgres`).
3. Reload the configuration after changes with `SELECT pg_reload_conf();`.
4. Test the settings with provided example queries.

---

## Memory-Related Parameters

These parameters control how PostgreSQL allocates and manages memory. Tuning them improves query performance and prevents resource bottlenecks.

### work_mem
- **Description**: Sets the maximum memory for internal operations (e.g., sorts, hashes) before writing to disk.
- **Default**: 4MB
- **Recommendation**: 1-2% of total system memory (e.g., 3-5 GB for 256 GB RAM).
- **Example Scenario**: System with 256 GB RAM, complex sorting queries.
```sql
-- Tuning work_mem for a system with 256 GB RAM
ALTER SYSTEM SET work_mem = '4GB';
SELECT pg_reload_conf();

-- Test query to observe impact
EXPLAIN ANALYZE SELECT * FROM large_table ORDER BY column1;
```
- Note: Higher values increase memory usage for sorting operations.

### shared_buffers
- **Description**: Amount of memory for caching database data.
- **Default**: 128MB
- **Recommendation**: 25-40% of total system memory (e.g., 64-102 GB for 256 GB RAM).
- **Example Scenario**: System with 256 GB RAM, heavy read/write workload.
```sql
-- Tuning shared_buffers for a system with 256 GB RAM
ALTER SYSTEM SET shared_buffers = '80GB';
SELECT pg_reload_conf();

-- Verify setting
SHOW shared_buffers;
```

### maintenance_work_mem
- **Description**: Memory for maintenance operations (e.g., VACUUM, CREATE INDEX).
- **Default**: 64MB
- **Recommendation**: 5-10% of total system memory (e.g., 13-26 GB for 256 GB RAM).
- **Example Scenario**: System with 256 GB RAM, frequent index rebuilds.
```sql
-- Tuning maintenance_work_mem for a system with 256 GB RAM
ALTER SYSTEM SET maintenance_work_mem = '20GB';
SELECT pg_reload_conf();

-- Test with VACUUM
VACUUM VERBOSE large_table;
```

---

## Parallelism-Related Parameters
These parameters control how tasks are divided across CPU cores, improving query performance by reducing execution time.

### max_parallel_maintenance_workers
- **Description**: Maximum parallel workers for maintenance operations (e.g., VACUUM, CREATE INDEX).
- **Default**: 2
- **Recommendation**: 12% of available cores (e.g., 6 cores for 48 cores).
- **Example Scenario**: System with 48 cores, large table maintenance.
```sql
-- Tuning max_parallel_maintenance_workers for 48 cores
ALTER SYSTEM SET max_parallel_maintenance_workers = 6;
SELECT pg_reload_conf();

-- Test with CREATE INDEX
CREATE INDEX idx_large_table ON large_table(column1);
```

### max_parallel_workers
- **Description**: Maximum parallel workers for query execution (e.g., scans, joins).
- **Default**: 8
- **Recommendation**: 75% of available cores (e.g., 36 cores for 48 cores).
- **Example Scenario**: System with 48 cores, complex queries.
```sql
-- Tuning max_parallel_workers for 48 cores
ALTER SYSTEM SET max_parallel_workers = 36;
SELECT pg_reload_conf();

-- Test parallel query
EXPLAIN ANALYZE SELECT * FROM large_table WHERE column1 > 1000;
```

### max_parallel_workers_per_gather
- **Description**: Maximum parallel workers per query operation (e.g., JOIN, SCAN).
- **Default**: 2
- **Recommendation**: 1/6 of total cores (e.g., 8 cores for 48 cores).
- **Example Scenario**: System with 48 cores, heavy JOIN operations.
```sql
-- Tuning max_parallel_workers_per_gather for 48 cores
ALTER SYSTEM SET max_parallel_workers_per_gather = 8;
SELECT pg_reload_conf();

-- Test with a join
EXPLAIN ANALYZE SELECT a.* FROM large_table a JOIN another_table b ON a.id = b.id;
```

### max_worker_processes
- **vDescription**: Maximum background worker processes for all tasks.
- **Default**: 8
- **Recommendation**: 100% of available cores (e.g., 48 cores for 48 cores).
- **Example Scenario**: System with 48 cores, mixed workloads.
```sql
-- Tuning max_worker_processes for 48 cores
ALTER SYSTEM SET max_worker_processes = 48;
SELECT pg_reload_conf();

-- Verify
SHOW max_worker_processes;
```

## JIT-Related Parameters
These parameters manage Just-In-Time (JIT) compilation to optimize query execution speed.

### jit
- **Description**: Enables/disables JIT compilation for CPU-bound queries.
- **Default**: on
- **Recommendation**: Enable (on) for CPU-bound workloads.
- **Example Scenario**: System with complex, CPU-intensive queries.
```sql
-- Enabling JIT for CPU-bound queries
ALTER SYSTEM SET jit = 'on';
SELECT pg_reload_conf();

-- Test with a complex query
EXPLAIN ANALYZE SELECT COUNT(*) FROM large_table WHERE complex_function(column1) > 50;
```
- Note: Aggressive settings may compile unnecessary query parts, increasing execution time.

### jit_above_cost
- **Description**: Minimum query cost for JIT compilation.
- **Default**: 100000
- **Recommendation**: Increase for selective use (e.g., 150000 for CPU-bound queries).
- **Example Scenario**: System with mixed query complexity.
```sql
-- Tuning jit_above_cost for selective JIT
ALTER SYSTEM SET jit_above_cost = 150000;
SELECT pg_reload_conf();

-- Test query
EXPLAIN ANALYZE SELECT * FROM large_table WHERE expensive_calculation(column1) > 100;
```

### jit_inline_above_cost
- **Description**: Minimum cost for inlining function calls with JIT.
- **Default**: 500000
- **Recommendation**: Adjust based on function complexity (e.g., 500000 as starting point).
- **Example Scenario**: System with frequent function calls.
```sql
-- Tuning jit_inline_above_cost
ALTER SYSTEM SET jit_inline_above_cost = 500000;
SELECT pg_reload_conf();

-- Test query
EXPLAIN ANALYZE SELECT custom_function(column1) FROM large_table;
```

### jit_optimize_above_cost
- **Description**: Cost threshold for additional JIT optimizations.
- **Default**: 500000
- **Recommendation**: Adjust for selective optimization (e.g., 500000 as starting point).
- **Example Scenario**: System with resource-heavy queries.
```sql
-- Tuning jit_optimize_above_cost
ALTER SYSTEM SET jit_optimize_above_cost = 500000;
SELECT pg_reload_conf();

-- Test query
EXPLAIN ANALYZE SELECT * FROM large_table WHERE complex_calculation(column1) > 200;
```

## Connection-Related Parameters
These parameters manage client connections, ensuring efficient handling under varying loads.

### max_connections
- **Description**: Maximum concurrent connections allowed.
- **Default**: 100
- **Recommendation**: (cores * 3) * 2 (e.g., 288 for 48 cores).
- **Example Scenario**: System with 48 cores, high connection demand.
```sql
-- Tuning max_connections for 48 cores
ALTER SYSTEM SET max_connections = 288;
SELECT pg_reload_conf();

-- Verify
SHOW max_connections;
```
- Note: Use a connection pooler if requirements exceed this value.

### idle_in_transaction_session_timeout
- **Description**: Time (ms) a session can remain idle in a transaction before termination.
- **Default**: 0 (disabled)
- **Recommendation**: 30-60 seconds (30000-60000 ms).
- **Example Scenario**: System with frequent idle transactions.
```sql
-- Setting timeout for idle transactions
ALTER SYSTEM SET idle_in_transaction_session_timeout = '30000';  -- 30 seconds
SELECT pg_reload_conf();

-- Test by leaving a transaction idle
BEGIN;
-- Wait 30+ seconds
SELECT * FROM pg_stat_activity WHERE state = 'idle in transaction';
```

### idle_session_timeout
- **Description**: Time (ms) a session can remain idle before termination.
- **Default**: 0 (disabled)
- **Recommendation**: 5-30 minutes (300000-1800000 ms).
- **Example Scenario**: System with inactive connections.
```sql
-- Setting timeout for idle sessions
ALTER SYSTEM SET idle_session_timeout = '300000';  -- 5 minutes
SELECT pg_reload_conf();

-- Verify active sessions
SELECT * FROM pg_stat_activity WHERE state = 'idle';
```

## Helper Scripts

### Apply Configuration
Reloads PostgreSQL configuration after changes.

```sql
-- Reload configuration
SELECT pg_reload_conf();
```

### Monitor Performance
Monitors sessions and resource usage.

```sql
-- Monitor active sessions and resource usage
SELECT pid, state, query, wait_event, pg_stat_activity.usename
FROM pg_stat_activity
WHERE state IS NOT NULL;

-- Check memory usage
SELECT name, setting FROM pg_settings 
WHERE name IN ('work_mem', 'shared_buffers', 'maintenance_work_mem');
```

## Additional Notes
- Testing: Replace large_table, column1, etc., with your actual table/column names.
- Hardware: Examples assume 256 GB RAM and 48 cores; adjust values for your system.
- Monitoring: Use EXPLAIN ANALYZE and pg_stat_activity to measure impact.
- Contributions: Feel free to fork this repo and submit pull requests with additional examples.
