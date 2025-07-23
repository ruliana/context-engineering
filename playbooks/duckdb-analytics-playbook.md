# DuckDB Analytics Playbook

## Key Concepts

- **DuckDB**: Open-source analytical database management system optimized for analytical workloads, supporting both in-memory and file-based operations
- **Analytical Database**: Column-oriented database designed for OLAP (Online Analytical Processing) rather than OLTP (Online Transaction Processing)
- **Vectorized Execution**: Query processing technique that operates on vectors/arrays of data rather than individual rows for better performance
- **Zero-Copy Integration**: Direct data access between DuckDB and Python data structures (Pandas, NumPy) without data copying
- **Lazy Evaluation**: Query optimization technique where operations are deferred until results are actually needed
- **Columnar Storage**: Data storage format where each column is stored separately, optimizing analytical queries
- **OLAP Cube**: Multidimensional data structure optimized for analytical queries with aggregations and grouping

## Common Patterns

### Installation and Setup
```python
# Install DuckDB
pip install duckdb

# Basic imports for data science workflow
import duckdb
import pandas as pd
import numpy as np
```

**When to use**: Always required for DuckDB operations
**Expected output**: Successful installation and import without errors

### Connection Patterns
```python
# In-memory database (default)
conn = duckdb.connect()

# Persistent database file
conn = duckdb.connect('analytics.duckdb')

# Context manager pattern (recommended)
with duckdb.connect('analytics.duckdb') as conn:
    result = conn.execute("SELECT 1").fetchall()
```

**When to use**: In-memory for temporary analysis, persistent for reusable datasets
**Expected output**: Connection object ready for queries

### Large File Import Pattern
```python
# Direct CSV querying without loading into memory
result = duckdb.execute("""
    SELECT COUNT(*), AVG(price), product_category
    FROM read_csv('large_dataset.csv')
    GROUP BY product_category
""").df()

# Parquet files (most efficient for analytics)
result = duckdb.execute("""
    SELECT * FROM read_parquet('data/*.parquet')
    WHERE date >= '2023-01-01'
""").df()
```

**When to use**: Files too large for Pandas memory constraints (>1GB)
**Expected output**: Pandas DataFrame with query results, no memory overflow

### Pandas Integration Pattern
```python
# Query Pandas DataFrame directly
df = pd.read_csv('small_file.csv')
result = duckdb.execute("""
    SELECT customer_id, SUM(amount) as total
    FROM df
    GROUP BY customer_id
    ORDER BY total DESC
    LIMIT 10
""").df()

# Convert DuckDB result to Pandas
conn = duckdb.connect()
conn.execute("CREATE TABLE sales AS SELECT * FROM read_csv('sales.csv')")
pandas_df = conn.execute("SELECT * FROM sales WHERE amount > 1000").df()
```

**When to use**: Need SQL capabilities on existing Pandas DataFrames or convert DuckDB results for ML pipelines
**Expected output**: Seamless DataFrame conversion and SQL query execution

## Implementation Details

### Environment Setup for Jupyter
```python
# Jupyter notebook cell 1
import duckdb
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

# Create persistent connection
conn = duckdb.connect('notebook_analysis.duckdb')
```

**Validation**: `conn.execute("SELECT 1").fetchall()` returns `[(1,)]`
**Common errors**: `ModuleNotFoundError` = run `pip install duckdb pandas matplotlib seaborn`

### Large Dataset Analysis Pattern
```python
# Efficient large file aggregation
conn.execute("""
    CREATE TABLE daily_metrics AS
    SELECT 
        date_trunc('day', timestamp) as date,
        COUNT(*) as events,
        AVG(value) as avg_value,
        PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY value) as p95_value
    FROM read_parquet('logs/*.parquet')
    WHERE timestamp >= '2023-01-01'
    GROUP BY date_trunc('day', timestamp)
    ORDER BY date
""")

# Extract for visualization
metrics_df = conn.execute("SELECT * FROM daily_metrics").df()
```

**Validation**: Query executes without memory errors, returns aggregated data
**Common errors**: `Out of Memory` = use `LIMIT` clause or smaller date ranges first

### ML Pipeline Integration
```python
# Feature engineering with DuckDB
features_df = conn.execute("""
    SELECT 
        customer_id,
        COUNT(*) as transaction_count,
        AVG(amount) as avg_amount,
        SUM(amount) as total_spent,
        MAX(amount) as max_amount,
        STDDEV(amount) as amount_stddev,
        DATE_DIFF('day', MIN(date), MAX(date)) as customer_tenure
    FROM transactions
    GROUP BY customer_id
    HAVING COUNT(*) >= 5
""").df()

# Prepare for ML training
features = features_df.drop(['customer_id'], axis=1)
target = features_df['total_spent']
features_train, features_test, target_train, target_test = train_test_split(features, target, test_size=0.2, random_state=42)

# Train model
model = RandomForestRegressor(n_estimators=100, random_state=42)
model.fit(features_train, target_train)
```

**Validation**: Features DataFrame has no null values, model training completes successfully
**Common errors**: `KeyError` = check column names match between SQL and Python

### Visualization Integration
```python
# Time series visualization
conn.execute("""
    CREATE VIEW monthly_trends AS
    SELECT 
        date_trunc('month', order_date) as month,
        SUM(amount) as revenue,
        COUNT(*) as orders,
        COUNT(DISTINCT customer_id) as unique_customers
    FROM orders
    GROUP BY date_trunc('month', order_date)
    ORDER BY month
""")

trend_data = conn.execute("SELECT * FROM monthly_trends").df()

# Create visualization
fig, axes = plt.subplots(2, 2, figsize=(15, 10))
trend_data.plot(x='month', y='revenue', ax=axes[0,0], title='Monthly Revenue')
trend_data.plot(x='month', y='orders', ax=axes[0,1], title='Monthly Orders')
trend_data.plot(x='month', y='unique_customers', ax=axes[1,0], title='Unique Customers')

# Correlation heatmap
corr_data = conn.execute("""
    SELECT revenue, orders, unique_customers 
    FROM monthly_trends
""").df()
sns.heatmap(corr_data.corr(), annot=True, ax=axes[1,1])
plt.tight_layout()
plt.show()
```

**Validation**: Plots display correctly, data aggregations are accurate
**Common errors**: `Empty DataFrame` = check date formats and data availability

### Performance Optimization Patterns
```python
# Index creation for repeated queries (ONLY for persistent tables)
conn.execute("CREATE INDEX idx_customer_date ON transactions(customer_id, date)")

# Parallel CSV reading with configuration
conn.execute("""
    SET threads TO 4;
    SET memory_limit = '4GB';
    SELECT COUNT(*) FROM read_csv('huge_file.csv', parallel=true)
""")

# Query optimization with EXPLAIN
plan = conn.execute("""
    EXPLAIN SELECT customer_id, SUM(amount)
    FROM transactions
    WHERE date >= '2023-01-01'
    GROUP BY customer_id
""").fetchall()
print(plan)
```

**Validation**: Queries execute faster with indices, parallel processing utilizes multiple cores
**Common errors**: `Memory limit exceeded` = reduce memory_limit or process data in chunks

### Indexing Capabilities and Costs

#### Table Indexes (Persistent and In-Memory)
```python
# ART (Adaptive Radix Tree) indexes for tables
conn.execute("CREATE TABLE sales (id INTEGER PRIMARY KEY, customer_id INTEGER, amount DOUBLE)")
conn.execute("CREATE INDEX idx_customer ON sales(customer_id)")  # Manual ART index
conn.execute("CREATE UNIQUE INDEX idx_unique_customer ON sales(customer_id)")  # Unique constraint

# Check index usage
conn.execute("EXPLAIN ANALYZE SELECT * FROM sales WHERE customer_id = 123")
```

**Index Types Available**:
- **Min-Max Index (Zonemap)**: Automatically created for all columns, tracks min/max values per data chunk
- **ART Index**: Created for PRIMARY KEY/UNIQUE constraints or manually with CREATE INDEX
- **Memory Cost**: ART indexes must fit entirely in memory during creation
- **Performance Trade-off**: Speeds up point queries and highly selective filters, slows down INSERT/UPDATE operations

#### CSV File Limitations
```python
# CSV files do NOT support traditional indexing
result = conn.execute("SELECT * FROM read_csv('large.csv') WHERE id = 123")  # Full scan required

# Workaround: Import to table first for indexing
conn.execute("""
    CREATE TABLE csv_indexed AS 
    SELECT * FROM read_csv('large.csv')
""")
conn.execute("CREATE INDEX idx_csv_id ON csv_indexed(id)")
```

**CSV Index Limitations**:
- **No Index Support**: Cannot create indexes directly on CSV files
- **Performance Impact**: Up to 600x slower than Parquet for analytical queries
- **File Size**: ~6.5x larger than Parquet files
- **Recommendation**: Convert large CSV files to Parquet or import to tables

#### Parquet File Optimization
```python
# Parquet files have built-in optimizations (no additional indexes needed)
result = conn.execute("""
    SELECT customer_id, SUM(amount) 
    FROM read_parquet('sales.parquet')
    WHERE date >= '2023-01-01'  -- Uses built-in statistics
    GROUP BY customer_id
""")

# Optimize Parquet row group size for parallelization
conn.execute("""
    COPY (SELECT * FROM large_table) 
    TO 'optimized.parquet' 
    WITH (FORMAT PARQUET, ROW_GROUP_SIZE 1000000)
""")
```

**Parquet Built-in Features**:
- **Zonemap Statistics**: Automatic min/max values per column chunk
- **Filter Pushdown**: DuckDB pushes WHERE clauses into file scan
- **Projection Pushdown**: Only reads requested columns
- **Optimal Row Group Size**: 100K-1M rows per group for best parallelization
- **Memory Efficiency**: ~10x less memory usage than DuckDB native format

#### In-Memory Database Index Costs
```python
# In-memory database with indexing
conn_memory = duckdb.connect()  # In-memory connection
conn_memory.execute("CREATE TABLE mem_table AS SELECT * FROM read_parquet('data.parquet')")
conn_memory.execute("CREATE INDEX idx_mem ON mem_table(customer_id)")

# Monitor memory usage
import psutil
process = psutil.Process()
memory_usage = process.memory_info().rss / 1024 / 1024  # MB
print(f"Memory usage: {memory_usage:.1f} MB")
```

**In-Memory Index Costs**:
- **Memory Overhead**: Can increase memory usage by 5-10x for large tables
- **Index Size**: ART indexes require significant RAM (must fit entirely in memory)
- **Performance Benefit**: 10-100x faster for point queries and highly selective filters
- **Recommendation**: Only create indexes on frequently queried columns

## Troubleshooting

### Memory Issues with Large Files
**Symptoms**: `std::bad_alloc`, `Out of memory` errors
**Cause**: Dataset exceeds available RAM or DuckDB memory limit
**Fix**: 
```python
# Set memory limit
conn.execute("SET memory_limit = '2GB'")

# Process in chunks
conn.execute("""
    CREATE TABLE results AS
    SELECT * FROM read_csv('large.csv')
    WHERE date >= '2023-01-01' AND date < '2023-02-01'
""")
```

### DataFrame Conversion Errors
**Symptoms**: `ArrowInvalid`, `TypeError` when calling `.df()`
**Cause**: Data type incompatibilities between DuckDB and Pandas
**Fix**:
```python
# Explicit type casting
result = conn.execute("""
    SELECT 
        customer_id::VARCHAR as customer_id,
        amount::DOUBLE as amount,
        date::DATE as date
    FROM transactions
""").df()
```

### Slow Query Performance
**Symptoms**: Queries taking much longer than expected
**Cause**: Missing indices, inefficient query patterns, or large result sets
**Fix**:
```python
# Add appropriate indices
conn.execute("CREATE INDEX idx_date ON sales(date)")

# Use LIMIT for exploration
conn.execute("SELECT * FROM large_table LIMIT 1000")

# Check query plan
conn.execute("EXPLAIN ANALYZE SELECT * FROM sales WHERE date > '2023-01-01'")
```

### File Format Issues
**Symptoms**: `Invalid Input Error`, parsing failures
**Cause**: Incorrect file format specification or corrupted data
**Fix**:
```python
# Specify CSV parameters explicitly
result = conn.execute("""
    SELECT * FROM read_csv('file.csv', 
        delimiter=';',
        header=true,
        quote='"',
        escape='"'
    )
""").df()
```

## Authoritative References

- [DuckDB Official Documentation](https://duckdb.org/docs/)
- [DuckDB Python API Guide](https://duckdb.org/docs/api/python/overview)
- [DuckDB SQL Introduction](https://duckdb.org/docs/sql/introduction)
- [DuckDB Data Import Documentation](https://duckdb.org/docs/data/overview)
- [DuckDB Performance Guide](https://duckdb.org/docs/guides/performance/overview)
- [DuckDB Extensions](https://duckdb.org/docs/extensions/overview)
