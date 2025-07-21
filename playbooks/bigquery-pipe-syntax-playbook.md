# BigQuery Pipe Syntax Playbook

## Key Concepts

- **Pipe Syntax**: Extension to GoogleSQL using `|>` operator for linear query structure
- **Pipe Operator**: Pattern `|> operator_name argument_list` that transforms data
- **Linear Query Structure**: Top-down data transformation approach replacing nested subqueries  
- **Standard SQL Compatibility**: Can mix pipe syntax with standard sql in same query
- **Default Availability**: Available to all BigQuery users by default starting February 4, 2025

## Common Patterns

### Basic Pipe Query Structure
```sql
from table_name
|> where condition
|> extend new_column as expression  
|> aggregate functions group by columns
```
**When to use**: For readable, step-by-step data transformations
**Expected output**: Clean, maintainable queries that flow logically

### WHERE Filtering Pattern
```sql
from mydataset.sales_data
|> where sale_date >= '2024-01-01'
|> where amount > 100
```
**When to use**: Multiple filtering conditions applied sequentially
**Expected output**: Progressively filtered result set

### EXTEND Column Creation Pattern
```sql
from mydataset.scores
|> extend (score1 + score2) / 2 as average_score
|> extend case when average_score >= 90 then 'A' else 'B' end as grade
```
**When to use**: Adding computed columns that reference previous columns
**Expected output**: Original table with additional calculated columns

### AGGREGATE Data Summarization Pattern
```sql
from mydataset.orders
|> aggregate sum(total) as revenue, count(*) as order_count 
   group by customer_id, product_category
```
**When to use**: Summarizing data without repeating GROUP BY columns
**Expected output**: Aggregated results grouped by specified columns

### SET Column Modification Pattern
```sql
from mydataset.products
|> set price = price * 1.1
|> set status = 'updated'
```
**When to use**: Modifying existing column values in-place
**Expected output**: Table with updated column values

### Column Management Patterns
```sql
-- drop columns
from mydataset.wide_table
|> drop unnecessary_column1, unnecessary_column2

-- rename columns  
from mydataset.raw_data
|> rename old_name as new_name
|> rename cryptic_col as descriptive_name
```
**When to use**: Cleaning up table structure
**Expected output**: Table with removed or renamed columns

## Implementation Details

### Converting Standard SQL to Pipe Syntax

#### Pattern 1: Nested Subqueries → Pipe Chain
**Standard SQL**:
```sql
select avg(monthly_sales), category
from (
  select sum(amount) as monthly_sales, category
  from sales 
  where date >= '2024-01-01'
  group by extract(month from date), category
)
group by category;
```

**Pipe Syntax**:
```sql
from sales
|> where date >= '2024-01-01'
|> extend extract(month from date) as month
|> aggregate sum(amount) as monthly_sales group by month, category  
|> aggregate avg(monthly_sales) group by category;
```

#### Pattern 2: Complex WHERE Conditions → Sequential Filters
**Standard SQL**:
```sql
select * from products 
where price > 100 
  and category in ('electronics', 'appliances')
  and stock_quantity > 0;
```

**Pipe Syntax**:
```sql
from products
|> where price > 100
|> where category in ('electronics', 'appliances') 
|> where stock_quantity > 0;
```

#### Pattern 3: Multiple CTEs → Pipe Operators
**Standard SQL**:
```sql
with filtered_data as (
  select * from orders where status = 'completed'
),
monthly_summary as (
  select customer_id, sum(total) as monthly_total
  from filtered_data 
  group by customer_id, extract(month from order_date)
)
select avg(monthly_total) from monthly_summary;
```

**Pipe Syntax**:
```sql
from orders
|> where status = 'completed'
|> extend extract(month from order_date) as month
|> aggregate sum(total) as monthly_total group by customer_id, month
|> aggregate avg(monthly_total);
```

### Converting Pipe Syntax to Standard SQL

#### Pattern 1: Pipe Chain → Nested Subqueries
**Pipe Syntax**:
```sql
from sales_data
|> where region = 'north'
|> extend profit_margin = (revenue - cost) / revenue
|> aggregate avg(profit_margin) group by product_category;
```

**Standard SQL**:
```sql
select product_category, avg(profit_margin)
from (
  select *, (revenue - cost) / revenue as profit_margin
  from sales_data
  where region = 'north'
)
group by product_category;
```

#### Pattern 2: Multiple Aggregations → Multiple Query Levels
**Pipe Syntax**:
```sql
from transactions  
|> aggregate count(*) as daily_count group by date(timestamp)
|> aggregate avg(daily_count) as avg_daily_transactions;
```

**Standard SQL**:
```sql
select avg(daily_count) as avg_daily_transactions
from (
  select date(timestamp), count(*) as daily_count
  from transactions
  group by date(timestamp)
);
```

### CTEs vs Pipe Operators: When to Use Each

#### CTEs Still Needed: Conceptual Organization
**Use CTEs for**: Logical separation of complex business logic, reusable intermediate results, or conceptual clarity
```sql
with customer_segments as (
  from customers
  |> extend case 
       when total_orders > 100 then 'vip'
       when total_orders > 10 then 'regular' 
       else 'new' 
     end as segment
),
segment_performance as (
  from sales join customer_segments using (customer_id)
  |> aggregate sum(amount) as revenue, count(*) as transactions
     group by segment
)
select * from segment_performance where revenue > 10000;
```

#### CTEs No Longer Needed: Replaced by Pipe Operators

**Pattern 1: Aggregation of Aggregations → Multiple AGGREGATE**
**Old CTE approach**:
```sql
with daily_sales as (
  select date(sale_timestamp) as sale_date, sum(amount) as daily_total
  from sales group by date(sale_timestamp)
),
monthly_averages as (
  select extract(month from sale_date) as month, avg(daily_total) as avg_daily
  from daily_sales group by extract(month from sale_date)
)
select * from monthly_averages;
```

**New Pipe approach**:
```sql
from sales
|> extend date(sale_timestamp) as sale_date
|> aggregate sum(amount) as daily_total group by sale_date
|> extend extract(month from sale_date) as month  
|> aggregate avg(daily_total) as avg_daily group by month;
```

**Pattern 2: Avoiding Repeated Complex Calculations → EXTEND**
**Old CTE approach**:
```sql
with enriched_data as (
  select *, 
    case when category = 'premium' then price * 0.9 else price end as final_price,
    (revenue - cost) / nullif(revenue, 0) as margin
  from products
)
select * from enriched_data 
where final_price > 100 and margin > 0.2;
```

**New Pipe approach**:
```sql
from products
|> extend case when category = 'premium' then price * 0.9 else price end as final_price
|> extend (revenue - cost) / nullif(revenue, 0) as margin
|> where final_price > 100 and margin > 0.2;
```

**Pattern 3: Window Functions + Filtering → EXTEND + WHERE**
**Old CTE with QUALIFY**:
```sql
with ranked_products as (
  select *,
    row_number() over (partition by category order by sales desc) as rn,
    avg(price) over (partition by category) as avg_category_price
  from products
  qualify rn <= 3
)
select * from ranked_products where price > avg_category_price;
```

**New Pipe approach**:
```sql
from products
|> extend row_number() over (partition by category order by sales desc) as rn
|> extend avg(price) over (partition by category) as avg_category_price  
|> where rn <= 3 and price > avg_category_price;
```

#### Decision Matrix: CTE vs Pipe Operators

| Use Case | Traditional CTE | Pipe Operators | Recommendation |
|----------|-----------------|----------------|----------------|
| Conceptual organization | ✓ | ✗ | Use CTE |
| Reusable named results | ✓ | ✗ | Use CTE |
| Aggregation of aggregations | ✓ | ✓ | Use Pipe (cleaner) |
| Complex calculated fields | ✓ | ✓ | Use Pipe (avoid repetition) |
| Window functions + filtering | ✓ | ✓ | Use Pipe (no QUALIFY needed) |
| Multi-step transformations | ✓ | ✓ | Use Pipe (more readable) |

### Accidental Complexity Patterns: Where Pipe Syntax Excels

#### Pattern 1: Nested Subquery Elimination  
**Accidental Complexity**: Traditional SQL requires inside-out reading of deeply nested subqueries
**Traditional SQL**:
```sql
select final_rank.*, category_stats.avg_price
from (
  select ranked_products.*, price_tier
  from (
    select *, row_number() over (partition by category order by sales desc) as sales_rank,
           case when price > 1000 then 'premium' else 'budget' end as price_tier
    from products where launch_date >= '2024-01-01'
  ) ranked_products
  where sales_rank <= 5
) final_rank
join (select category, avg(price) as avg_price from products group by category) category_stats 
  using (category);
```

**Pipe Syntax**:
```sql
from products
|> where launch_date >= '2024-01-01'
|> extend case when price > 1000 then 'premium' else 'budget' end as price_tier
|> extend row_number() over (partition by category order by sales desc) as sales_rank
|> where sales_rank <= 5
|> join (from products |> aggregate avg(price) as avg_price group by category) using (category);
```

#### Pattern 2: Redundant Expression Elimination
**Accidental Complexity**: Repeating complex calculations in SELECT, WHERE, and ORDER BY clauses
**Traditional SQL**:
```sql
select product_id, (revenue - cost) / nullif(revenue, 0) * 100 as profit_margin_pct,
       case when (revenue - cost) / nullif(revenue, 0) * 100 > 30 then 'high_margin' else 'low_margin' end
from products
where (revenue - cost) / nullif(revenue, 0) * 100 > 10
order by (revenue - cost) / nullif(revenue, 0) * 100 desc;
```

**Pipe Syntax**:
```sql
from products
|> extend (revenue - cost) / nullif(revenue, 0) * 100 as profit_margin_pct
|> where profit_margin_pct > 10
|> extend case when profit_margin_pct > 30 then 'high_margin' else 'low_margin' end as margin_category
|> order by profit_margin_pct desc;
```

#### Pattern 3: Complex JOIN Pre-filtering
**Accidental Complexity**: JOINing large tables then filtering creates unnecessary computation
**Traditional SQL**:
```sql
select u.user_id, count(distinct o.order_id) as order_count, sum(oi.quantity * p.price) as total_spent
from users u
left join orders o on u.user_id = o.user_id and o.order_date >= '2024-01-01' and o.status = 'completed'
left join order_items oi on o.order_id = oi.order_id
left join products p on oi.product_id = p.product_id and p.category in ('electronics', 'appliances')
where u.registration_date >= '2023-01-01' and u.status = 'active'
group by u.user_id
having count(distinct o.order_id) > 0;
```

**Pipe Syntax**:
```sql
from users
|> where registration_date >= '2023-01-01' and status = 'active'
|> join (from orders |> where order_date >= '2024-01-01' and status = 'completed') using (user_id)
|> join (
     from order_items oi
     |> join (from products |> where category in ('electronics', 'appliances')) p using (product_id)
     |> extend quantity * price as line_total
   ) using (order_id)
|> aggregate count(distinct order_id) as order_count, sum(line_total) as total_spent group by user_id
|> where order_count > 0;
```

#### Pattern 4: Window Function Repetition Reduction
**Accidental Complexity**: Multiple window functions with repetitive OVER clauses
**Traditional SQL**:
```sql
select product_id, sales, price,
       row_number() over (partition by category order by sales desc) as category_sales_rank,
       lag(sales, 1) over (partition by category order by launch_date) as prev_product_sales,
       avg(sales) over (partition by category) as category_avg_sales,
       sales - avg(sales) over (partition by category) as sales_vs_category_avg
from products where status = 'active';
```

**Pipe Syntax**:
```sql
from products
|> where status = 'active'
|> extend row_number() over (partition by category order by sales desc) as category_sales_rank
|> extend lag(sales, 1) over (partition by category order by launch_date) as prev_product_sales
|> extend avg(sales) over (partition by category) as category_avg_sales
|> extend sales - category_avg_sales as sales_vs_category_avg;
```

#### Pattern 5: Complex Conditional Logic Flattening
**Accidental Complexity**: Deeply nested CASE statements for business rules
**Traditional SQL**:
```sql
select order_id,
  case when customer_segment = 'vip' then
    case when order_total > 1000 then 
      case when shipping_country = 'US' then 0.0
           when shipping_country in ('CA', 'MX') then order_total * 0.05
           else order_total * 0.10 end
    else order_total * 0.02 end
  when customer_segment = 'premium' then
    case when order_total > 500 then order_total * 0.03 else order_total * 0.05 end
  else order_total * 0.08 end as shipping_cost
from orders;
```

**Pipe Syntax**:
```sql
from orders
|> extend case 
     when customer_segment = 'vip' and order_total > 1000 and shipping_country = 'US' then 0.0
     when customer_segment = 'vip' and order_total > 1000 and shipping_country in ('CA', 'MX') then order_total * 0.05
     when customer_segment = 'vip' and order_total > 1000 then order_total * 0.10
     when customer_segment = 'vip' then order_total * 0.02
     when customer_segment = 'premium' and order_total > 500 then order_total * 0.03
     when customer_segment = 'premium' then order_total * 0.05
     else order_total * 0.08
   end as shipping_cost;
```

#### Pattern 6: Data Type Conversion Pipeline Clarification
**Accidental Complexity**: Nested type conversions with conditional logic
**Traditional SQL**:
```sql
select customer_id,
  case when customer_type = 'enterprise' then 
    case when safe_cast(annual_revenue as float64) > 1000000 then 'tier_1'
         when safe_cast(annual_revenue as float64) > 100000 then 'tier_2'
         else 'tier_3' end
  else 'unclassified' end as customer_tier
from customers
where safe_cast(annual_revenue as float64) is not null;
```

**Pipe Syntax**:
```sql
from customers
|> extend safe_cast(annual_revenue as float64) as revenue_numeric
|> where revenue_numeric is not null
|> extend case 
     when customer_type = 'enterprise' and revenue_numeric > 1000000 then 'tier_1'
     when customer_type = 'enterprise' and revenue_numeric > 100000 then 'tier_2'
     when customer_type = 'enterprise' then 'tier_3'
     else 'unclassified'
   end as customer_tier;
```

#### Key Benefits of Pipe Syntax for Complex Queries
- **Linear Logic Flow**: Top-down reading matches natural thought process
- **Expression Reuse**: EXTEND eliminates repeated calculations
- **Independent Testing**: Each pipe step can be validated separately  
- **Reduced Nesting**: Eliminates inside-out subquery reading
- **Performance Optimization**: Early filtering and cleaner join logic
- **Maintenance Simplicity**: Changes require fewer code modifications

### Advanced Patterns

#### Mixing Standard SQL with Pipe Syntax
```sql
select final_result.*, category_rank 
from (
  from sales_data
  |> where sale_date >= '2024-01-01'
  |> aggregate sum(amount) as total_sales group by product_id
  |> where total_sales > 1000
) as final_result
join product_rankings on final_result.product_id = product_rankings.product_id;
```

#### Complex Data Pipeline
```sql
from raw_events
|> where event_type in ('page_view', 'click', 'purchase')
|> extend date(timestamp) as event_date
|> extend extract(hour from timestamp) as event_hour  
|> aggregate count(*) as event_count group by user_id, event_date, event_hour
|> where event_count > 5
|> aggregate avg(event_count) as avg_hourly_events group by user_id;
```

**Validation**: Query executes without syntax errors and returns expected aggregated results
**Common errors**: 
- `Column not found` = Ensure EXTEND columns are defined before use
- `GROUP BY required` = Add GROUP BY when using aggregate functions

## Troubleshooting

### Error Pattern: Column Reference Issues
**Symptoms**: `Column 'column_name' not found` or `Column must be grouped`
**Cause**: Referencing columns created in later pipe steps or missing GROUP BY
**Fix**: Use EXTEND to create columns before referencing, ensure proper GROUP BY clauses

### Error Pattern: Aggregate Function Misuse  
**Symptoms**: `Aggregate function calls cannot be nested` or `Column must appear in GROUP BY`
**Cause**: Incorrect aggregation syntax or missing GROUP BY columns
**Fix**: 
```sql
-- Correct pattern
|> aggregate sum(amount) as total, count(*) as count group by category
-- Not: |> aggregate sum(count(amount)) group by category
```

### Error Pattern: Pipe Operator Order
**Symptoms**: Unexpected results or performance issues
**Cause**: Inefficient pipe operator ordering
**Fix**: Place WHERE filters early, use EXTEND before AGGREGATE:
```sql
from table
|> where condition          -- Filter first
|> extend calculated_field  -- Add columns next  
|> aggregate functions      -- Aggregate last
```

### Error Pattern: Standard SQL Mixing Issues
**Symptoms**: Syntax errors when combining pipe and standard SQL
**Cause**: Incorrect subquery structure or operator precedence
**Fix**: Wrap pipe syntax in parentheses when using as subquery:
```sql
select * from (
  from table |> where condition |> extend new_col
) as pipe_result;
```

### Error Pattern: JOIN with Pipe Syntax
**Symptoms**: JOIN syntax errors or unexpected results  
**Cause**: Incorrect JOIN operator usage in pipe context
**Fix**: Use standard JOIN syntax or pipe JOIN operator:
```sql
-- Method 1: Standard join after pipe
from (
  from table1 |> where condition  
) t1
join table2 on t1.id = table2.id;

-- Method 2: Pipe join operator  
from table1
|> join table2 on table1.id = table2.id
|> where condition;
```

## Authoritative References

- [Work with pipe query syntax | BigQuery | Google Cloud](https://cloud.google.com/bigquery/docs/pipe-syntax-guide)
- [Pipe query syntax | BigQuery | Google Cloud](https://cloud.google.com/bigquery/docs/reference/standard-sql/pipe-syntax)
- [Pipe syntax | BigQuery | Google Cloud](https://cloud.google.com/bigquery/docs/pipe-syntax)
- [Simplify your SQL with pipe syntax in BigQuery | Google Cloud Blog](https://cloud.google.com/blog/products/data-analytics/simplify-your-sql-with-pipe-syntax-in-bigquery-and-cloud-logging)
- [Introducing pipe syntax in BigQuery | Google Cloud Blog](https://cloud.google.com/blog/products/data-analytics/introducing-pipe-syntax-in-bigquery-and-cloud-logging)