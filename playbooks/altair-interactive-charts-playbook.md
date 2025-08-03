# Altair Interactive Charts Playbook

## Key Concepts

- **Declarative Visualization**: Describe what you want, not how to draw it - Altair handles rendering details
- **Grammar of Graphics**: Bind data columns to visual encoding channels (x, y, color, size) using Vega-Lite grammar
- **Selection Objects**: Interactive elements that capture user input (`alt.selection_point()`, `alt.selection_interval()`)
- **Parameter Binding**: Connect chart parameters to UI widgets (sliders, dropdowns, checkboxes) for dynamic control
- **Conditional Encoding**: Use `alt.when().then().otherwise()` to change visual properties based on selections or parameters
- **Transform Chain**: Apply data transformations (filter, calculate, aggregate) that respond to user interactions
- **Reactive Updates**: Charts automatically re-render when underlying data or parameters change
- **Marimo Integration**: Use `mo.ui.altair_chart()` to embed interactive Altair charts in reactive Marimo notebooks
- **Chart Value Access**: Selected data becomes available through `.value` property in subsequent Marimo cells
- **Data Sources**: Works with pandas DataFrames and DuckDB query results for local data analysis

## Common Patterns

### Basic Interactive Chart with Marimo
```python
import marimo as mo
import altair as alt
import pandas as pd
import numpy as np

# Create sample car dataset
_sample_data = pd.DataFrame({
    'horsepower': np.random.randint(70, 300, 100),
    'mpg': 40 - np.random.exponential(8, 100),
    'origin': np.random.choice(['USA', 'Europe', 'Japan'], 100),
    'name': [f'Car_{i}' for i in range(100)]
})
_sample_data['mpg'] = np.clip(_sample_data['mpg'], 10, 45)  # Realistic MPG range

car_data = _sample_data.copy()

# Create base chart with selection
_brush = alt.selection_interval(encodings=['x', 'y'])
_chart = alt.Chart(car_data).mark_circle(size=60).add_params(_brush).encode(
    x='horsepower:Q',
    y='mpg:Q',
    color=alt.when(_brush).then('origin:N').otherwise(alt.value('lightgray')),
    tooltip=['name:N', 'horsepower:Q', 'mpg:Q', 'origin:N']
).properties(
    width=500,
    height=300,
    title="Interactive Car Data Explorer"
)

# Wrap in Marimo UI element
interactive_chart = mo.ui.altair_chart(_chart)
interactive_chart
```
**When to use**: Basic data exploration with selection feedback
**Expected output**: Chart with brushable selection, gray points outside selection

### Parameter-Driven Filtering with Widgets
```python
# Cell 1: Define parameters and widgets
year_range = mo.ui.range_slider(
    start=1970, stop=2020, value=[1980, 2000], 
    label="Year Range"
)
origin_filter = mo.ui.multiselect(
    options=['USA', 'Europe', 'Japan'], 
    value=['USA'], 
    label="Origins"
)
mpg_threshold = mo.ui.slider(
    start=10, stop=50, value=20, 
    label="Minimum MPG"
)

mo.hstack([year_range, origin_filter, mpg_threshold])
```

```python
# Cell 2: Apply filters to data and create chart
_start_year, _end_year = year_range.value
_selected_origins = origin_filter.value
_min_mpg = mpg_threshold.value

# Add year column to car_data for filtering
_extended_data = car_data.copy()
_extended_data['year'] = np.random.randint(_start_year, _end_year + 1, len(car_data))

# Filter data with private variables
_filtered_data = _extended_data[
    (_extended_data['year'] >= _start_year) & 
    (_extended_data['year'] <= _end_year) &
    (_extended_data['origin'].isin(_selected_origins)) &
    (_extended_data['mpg'] >= _min_mpg)
]

_chart = alt.Chart(_filtered_data).mark_point(size=100).encode(
    x=alt.X('horsepower:Q', title='Horsepower'),
    y=alt.Y('mpg:Q', title='Miles per Gallon'),
    color=alt.Color('origin:N', title='Origin'),
    tooltip=['name:N', 'year:O', 'horsepower:Q', 'mpg:Q']
).properties(
    width=600,
    height=400,
    title=f"Cars: {len(_filtered_data)} vehicles matching criteria"
)

filtered_chart = mo.ui.altair_chart(_chart)
filtered_chart
```
**When to use**: Dashboard-style filtering with immediate visual feedback
**Expected output**: Chart updates automatically as widget values change

### Real-time ML Training Monitoring
```python
# Cell 1: Setup training metrics tracking
training_metrics = mo.ui.button(label="Start Training", kind="success")
training_metrics
```

```python
# Cell 2: Simulate ML training with live metrics
mo.stop(not training_metrics.value, mo.md("👆 Click to start training simulation"))

import numpy as np
import time

# Simulate training epochs with metrics
_epochs = []
_losses = []
_precisions = []
_recalls = []

# Private simulation loop
for _epoch in range(1, 21):
    _loss = 2.0 * np.exp(-_epoch / 10) + 0.1 * np.random.random()
    _precision = 0.95 * (1 - np.exp(-_epoch / 8)) + 0.05 * np.random.random()
    _recall = 0.90 * (1 - np.exp(-_epoch / 12)) + 0.1 * np.random.random()
    
    _epochs.append(_epoch)
    _losses.append(_loss)
    _precisions.append(_precision)
    _recalls.append(_recall)

# Create metrics DataFrame
_metrics_df = pd.DataFrame({
    'epoch': _epochs,
    'loss': _losses,
    'precision': _precisions,
    'recall': _recalls
})

# Transform for visualization
_melted_metrics = _metrics_df.melt(
    id_vars=['epoch'], 
    value_vars=['precision', 'recall'],
    var_name='metric',
    value_name='score'
)

# Loss chart
_loss_chart = alt.Chart(_metrics_df).mark_line(
    point=True, strokeWidth=3
).encode(
    x=alt.X('epoch:O', title='Epoch'),
    y=alt.Y('loss:Q', title='Training Loss', scale=alt.Scale(type='log')),
    color=alt.value('red')
).properties(
    width=600,
    height=200,
    title="Training Loss Over Time"
)

# Metrics chart
_metrics_chart = alt.Chart(_melted_metrics).mark_line(
    point=True, strokeWidth=3
).encode(
    x=alt.X('epoch:O', title='Epoch'),
    y=alt.Y('score:Q', title='Score', scale=alt.Scale(domain=[0, 1])),
    color=alt.Color('metric:N', title='Metric'),
    tooltip=['epoch:O', 'metric:N', 'score:Q']
).properties(
    width=600,
    height=200,
    title="Precision and Recall Over Time"
)

# Combined dashboard
_combined_chart = alt.vconcat(_loss_chart, _metrics_chart)

training_dashboard = mo.ui.altair_chart(_combined_chart)
training_dashboard
```
**When to use**: ML model training visualization and monitoring
**Expected output**: Real-time updating charts showing loss decay and metric improvement

### Linked Multi-Chart Dashboard
```python
# Cell 1: Create coordinated selection across multiple charts
# Create sample time series data
_dates = pd.date_range('2024-01-01', periods=365, freq='D')
_categories = ['Sales', 'Marketing', 'Operations']
time_series_data = pd.DataFrame({
    'date': np.tile(_dates, len(_categories)),
    'value': np.concatenate([np.random.randn(len(_dates)).cumsum() + i*50 for i in range(len(_categories))]),
    'category': np.repeat(_categories, len(_dates))
})
time_series_data['value'] = np.abs(time_series_data['value'])  # Ensure positive values

_brush_selection = alt.selection_interval(encodings=['x'])

# Overview chart with brush selection
_overview_chart = alt.Chart(time_series_data).mark_area(
    opacity=0.7
).add_params(_brush_selection).encode(
    x=alt.X('date:T', title='Date'),
    y=alt.Y('value:Q', title='Total Value'),
    color=alt.value('steelblue')
).properties(
    width=600,
    height=100,
    title="Overview - Brush to Filter Detail View"
)

# Detail chart filtered by brush
_detail_chart = alt.Chart(time_series_data).mark_line(
    point=True
).encode(
    x=alt.X('date:T', title='Date', scale=alt.Scale(domain=_brush_selection)),
    y=alt.Y('value:Q', title='Value'),
    color='category:N',
    tooltip=['date:T', 'value:Q', 'category:N']
).properties(
    width=600,
    height=300,
    title="Detailed View - Filtered by Brush Selection"
)

# Category breakdown chart also filtered by brush
_category_chart = alt.Chart(time_series_data).mark_bar().transform_filter(
    _brush_selection
).transform_aggregate(
    total_value='sum(value)',
    groupby=['category']
).encode(
    x=alt.X('total_value:Q', title='Total Value'),
    y=alt.Y('category:N', title='Category', sort='-x'),
    color='category:N',
    tooltip=['category:N', 'total_value:Q']
).properties(
    width=300,
    height=200,
    title="Category Totals"
)

# Combine all charts
_dashboard = alt.vconcat(
    _overview_chart,
    alt.hconcat(_detail_chart, _category_chart)
)

linked_dashboard = mo.ui.altair_chart(_dashboard)
linked_dashboard
```
**When to use**: Complex data exploration with coordinated views
**Expected output**: Brush selection in overview updates both detail and category charts

### DuckDB Integration with Dynamic Queries
```python
# Cell 1: Setup DuckDB connection and base query controls
import duckdb
import os

# Create sample database if it doesn't exist
_db_path = 'analytics.db'
if not os.path.exists(_db_path):
    _conn = duckdb.connect(_db_path)
    # Create sample transactions table
    _sample_transactions = pd.DataFrame({
        'date': pd.date_range('2023-01-01', '2023-12-31', freq='D'),
        'amount': np.random.exponential(200, 365),
        'category': np.random.choice(['sales', 'marketing', 'operations'], 365),
        'region': np.random.choice(['North', 'South', 'East', 'West'], 365)
    })
    _conn.execute("CREATE TABLE transactions AS SELECT * FROM _sample_transactions")
    _conn.close()

db_query_controls = mo.ui.form({
    "start_date": mo.ui.date(value="2023-01-01", label="Start Date"),
    "end_date": mo.ui.date(value="2023-12-31", label="End Date"),
    "min_amount": mo.ui.number(value=100, label="Minimum Amount"),
    "category": mo.ui.dropdown(
        options=["all", "sales", "marketing", "operations"], 
        value="all", 
        label="Category"
    )
})

db_query_controls
```

```python
# Cell 2: Execute dynamic DuckDB query and visualize
mo.stop(db_query_controls.value is None, mo.md("Submit query parameters"))

_query_params = db_query_controls.value
_conn = duckdb.connect('analytics.db')

# Build dynamic query with private variables
_base_query = """
    SELECT date, amount, category, region
    FROM transactions 
    WHERE date >= ? AND date <= ? AND amount >= ?
"""

_params = [
    _query_params["start_date"],
    _query_params["end_date"], 
    _query_params["min_amount"]
]

if _query_params["category"] != "all":
    _base_query += " AND category = ?"
    _params.append(_query_params["category"])

# Execute query and get results
_query_result = _conn.execute(_base_query, _params).fetchdf()

# Create interactive visualization
_scatter_plot = alt.Chart(_query_result).mark_circle(size=60).encode(
    x=alt.X('date:T', title='Date'),
    y=alt.Y('amount:Q', title='Amount'),
    color=alt.Color('category:N', title='Category'),
    size=alt.value(60),
    tooltip=['date:T', 'amount:Q', 'category:N', 'region:N']
).properties(
    width=700,
    height=400,
    title=f"Transaction Analysis: {len(_query_result)} records"
).interactive()

duckdb_chart = mo.ui.altair_chart(_scatter_plot)
duckdb_chart
```
**When to use**: Database-driven interactive analytics with dynamic querying
**Expected output**: Chart updates based on form parameters, showing database query results

## Implementation Details

### Installation and Setup
1. **Install required packages**:
   ```bash
   pip install "altair[all]" marimo pandas duckdb
   ```

2. **Enable Altair in Marimo**:
   ```python
   import marimo as mo
   import altair as alt
   import pandas as pd
   
   # Configure Altair renderer for notebooks
   alt.data_transformers.enable('json')
   ```

3. **Verify installation**:
   ```python
   # Test basic chart creation
   _test_data = pd.DataFrame({'x': [1, 2, 3], 'y': [4, 5, 6]})
   _test_chart = alt.Chart(_test_data).mark_point().encode(x='x', y='y')
   test_display = mo.ui.altair_chart(_test_chart)
   test_display
   ```

**Validation**: Chart renders in Marimo interface with interactive capabilities

### Advanced Selection Patterns

#### Multi-Chart Coordination with Shared Parameters
```python
# Cell 1: Define shared parameters
shared_filter = alt.param(
    name='category_filter',
    value='all',
    bind=alt.binding_select(
        options=['all', 'A', 'B', 'C'],
        name='Category Filter: '
    )
)

date_brush = alt.selection_interval(encodings=['x'], name='date_range')
```

#### Complex Conditional Encodings
```python
# Cell 2: Apply parameters across multiple visual properties
_base_chart = alt.Chart(car_data).add_params(shared_filter, date_brush)

_filtered_chart = _base_chart.transform_filter(
    f'(datum.category == "{shared_filter.name}") | ("{shared_filter.name}" == "all")'
).mark_circle().encode(
    x=alt.X('date:T', scale=alt.Scale(domain=date_brush)),
    y='value:Q',
    size=alt.when(date_brush).then(alt.value(100)).otherwise(alt.value(50)),
    color=alt.when(date_brush).then('category:N').otherwise(alt.value('lightgray')),
    opacity=alt.when(date_brush).then(alt.value(1.0)).otherwise(alt.value(0.3)),
    tooltip=['date:T', 'value:Q', 'category:N']
).properties(width=600, height=400)

advanced_chart = mo.ui.altair_chart(_filtered_chart)
advanced_chart
```

### Performance Optimization for Large Datasets

#### Data Sampling and Aggregation
```python
# Handle large datasets efficiently - create simulated large dataset
_large_data = pd.DataFrame({
    'timestamp': pd.date_range('2024-01-01', periods=100000, freq='1min'),
    'value': np.random.randn(100000).cumsum(),
    'category': np.random.choice(['A', 'B', 'C', 'D'], 100000)
})

# Sample for interactive exploration
_sample_size = 5000
_sampled_data = _large_data.sample(n=_sample_size, random_state=42)

# Aggregated view for overview
_hourly_agg = _large_data.groupby([
    pd.Grouper(key='timestamp', freq='H'),
    'category'
]).agg({
    'value': ['mean', 'sum', 'count']
}).reset_index()

_hourly_agg.columns = ['timestamp', 'category', 'avg_value', 'total_value', 'count']

# Create dual-resolution chart
_overview = alt.Chart(_hourly_agg).mark_line().encode(
    x='timestamp:T',
    y='avg_value:Q',
    color='category:N'
).properties(width=600, height=200, title="Hourly Aggregation")

_detail = alt.Chart(_sampled_data).mark_point(size=30).encode(
    x='timestamp:T',
    y='value:Q',
    color='category:N',
    tooltip=['timestamp:T', 'value:Q', 'category:N']
).properties(width=600, height=300, title=f"Sample Detail ({_sample_size} points)")

performance_chart = mo.ui.altair_chart(alt.vconcat(_overview, _detail))
performance_chart
```

### Real-time Data Streaming Simulation

#### Incremental Data Updates with Button Triggers
```python
# Cell 1: Initialize streaming data simulation
# Use global variable for data that needs to persist across cells
streaming_data = pd.DataFrame({
    'timestamp': pd.date_range('2024-01-01', periods=50, freq='1min'),
    'value': np.random.randn(50).cumsum(),
    'category': np.random.choice(['A', 'B', 'C'], 50)
})

stream_control = mo.ui.button(label="Add New Data Point", kind="success")
reset_stream = mo.ui.button(label="Reset Stream", kind="danger")

mo.hstack([stream_control, reset_stream])
```

```python
# Cell 2: Handle streaming updates
import datetime

# Use private variables for temporary calculations, update global data
if reset_stream.value:
    streaming_data = pd.DataFrame({
        'timestamp': pd.date_range('2024-01-01', periods=50, freq='1min'),
        'value': np.random.randn(50).cumsum(),
        'category': np.random.choice(['A', 'B', 'C'], 50)
    })

if stream_control.value:
    _last_timestamp = streaming_data['timestamp'].max()
    _last_value = streaming_data['value'].iloc[-1]
    
    _new_point = pd.DataFrame({
        'timestamp': [_last_timestamp + pd.Timedelta(minutes=1)],
        'value': [_last_value + np.random.randn()],
        'category': [np.random.choice(['A', 'B', 'C'])]
    })
    
    streaming_data = pd.concat([streaming_data, _new_point], ignore_index=True)
    
    # Keep only last 100 points for performance
    if len(streaming_data) > 100:
        streaming_data = streaming_data.tail(100).reset_index(drop=True)

# Create real-time chart
_stream_chart = alt.Chart(streaming_data).mark_line(point=True).encode(
    x=alt.X('timestamp:T', title='Time'),
    y=alt.Y('value:Q', title='Value'),
    color=alt.Color('category:N', title='Category'),
    tooltip=['timestamp:T', 'value:Q', 'category:N']
).properties(
    width=700,
    height=400,
    title=f"Live Data Stream ({len(streaming_data)} points)"
)

streaming_chart = mo.ui.altair_chart(_stream_chart)
streaming_chart
```

## Validation Methods

### Selection Data Access Testing
```python
# Test chart selections in separate cell from chart definition
_selected_data = interactive_chart.value
_selection_summary = {
    'count': len(_selected_data),
    'columns': list(_selected_data.columns) if len(_selected_data) > 0 else [],
    'has_data': len(_selected_data) > 0
}

mo.md(f"""
**Selection Validation:**
- Selected points: {_selection_summary['count']}
- Available columns: {_selection_summary['columns']}
- Data available: {_selection_summary['has_data']}
""")
```

### Widget Integration Verification
```python
# Verify widget values update chart data
_widget_values = {
    'year_range': year_range.value,
    'origins': origin_filter.value,
    'mpg_threshold': mpg_threshold.value
}

_filtered_count = len(_filtered_data)
_total_count = len(_extended_data)

mo.md(f"""
**Filter Validation:**
- Year range: {_widget_values['year_range']}
- Selected origins: {_widget_values['origins']}
- MPG threshold: {_widget_values['mpg_threshold']}
- Filtered records: {_filtered_count}/{_total_count} ({_filtered_count/_total_count*100:.1f}%)
""")
```

### Performance Monitoring
```python
import time

# Time chart rendering performance
_start_time = time.time()
_performance_chart = alt.Chart(_large_data).mark_point().encode(
    x='value:Q', y='timestamp:T', color='category:N'
)
_render_time = time.time() - _start_time

mo.md(f"Chart rendering time: {_render_time:.3f} seconds")

# Data size recommendations
_data_size = len(_large_data)
_recommendation = "✅ Good" if _data_size < 5000 else "⚠️ Consider sampling" if _data_size < 50000 else "❌ Sampling required"

mo.md(f"Data size: {_data_size} rows - {_recommendation}")
```

## Anti-patterns to Avoid

### ❌ Accessing Chart Values in Same Cell
```python
# DON'T DO THIS - won't work in Marimo
chart = mo.ui.altair_chart(alt.Chart(car_data).mark_point())
selected_data = chart.value  # ERROR: value not available yet
```

### ✅ Separate Chart Definition and Value Access
```python
# Cell 1: Define chart
chart = mo.ui.altair_chart(alt.Chart(car_data).mark_point().encode(x='horsepower:Q', y='mpg:Q'))
chart

# Cell 2: Access values
_selected_data = chart.value
processed_selection = _selected_data.copy() if len(_selected_data) > 0 else pd.DataFrame()
```

### ❌ Using Global Variables for Chart Data
```python
# DON'T DO THIS - breaks reactivity
filtered_data = car_data[car_data['mpg'] > threshold.value]  # Global variable
chart = alt.Chart(filtered_data).mark_point()
```

### ✅ Use Private Variables for Data Processing
```python
# DO THIS - maintains reactivity
_threshold_value = threshold.value
_filtered_data = car_data[car_data['mpg'] > _threshold_value]
chart_data = _filtered_data.copy()

_chart = alt.Chart(chart_data).mark_point().encode(x='horsepower:Q', y='mpg:Q')
reactive_chart = mo.ui.altair_chart(_chart)
```

### ❌ Creating Charts Without Selections for Interactivity
```python
# DON'T DO THIS - misses interactive potential
static_chart = alt.Chart(car_data).mark_point().encode(x='horsepower:Q', y='mpg:Q')
```

### ✅ Add Selections for Rich Interactivity
```python
# DO THIS - enables user interaction
_brush = alt.selection_interval()
_chart = alt.Chart(car_data).mark_point().add_params(_brush).encode(
    x='horsepower:Q',
    y='mpg:Q',
    color=alt.when(_brush).then('origin:N').otherwise(alt.value('gray'))
)
interactive_chart = mo.ui.altair_chart(_chart)
```

### ❌ Inefficient Large Dataset Handling
```python
# DON'T DO THIS - performance issues with large data
huge_chart = alt.Chart(million_row_dataset).mark_point().encode(x='value:Q', y='timestamp:T')
```

### ✅ Use Sampling and Aggregation
```python
# DO THIS - handle large data efficiently
_sample_data = million_row_dataset.sample(n=10000, random_state=42)
_chart = alt.Chart(_sample_data).mark_point().encode(
    x='value:Q', 
    y='timestamp:T',
    tooltip=['value:Q', 'timestamp:T', 'category:N']
).properties(title=f"Sample View (10k of {len(million_row_dataset)} points)")

efficient_chart = mo.ui.altair_chart(_chart)
```

## Debugging and Troubleshooting

### Chart Not Rendering Issues

#### Empty or Invalid Data
**Symptoms**: Chart area appears but no marks are visible
**Cause**: Data contains null values or wrong data types
**Fix**: Clean and validate data before charting

```python
# Debug data issues
_data_info = {
    'shape': car_data.shape,
    'dtypes': car_data.dtypes.to_dict(),
    'null_counts': car_data.isnull().sum().to_dict(),
    'sample': car_data.head().to_dict()
}

mo.md(f"""
**Data Debug Info:**
- Shape: {_data_info['shape']}
- Column types: {_data_info['dtypes']}
- Null values: {_data_info['null_counts']}
""")

# Clean data
_cleaned_data = car_data.dropna().copy()
_cleaned_data['x'] = pd.to_numeric(_cleaned_data['x'], errors='coerce')
_cleaned_data = _cleaned_data.dropna()

cleaned_chart_data = _cleaned_data
```

#### Scale Domain Issues
**Symptoms**: Chart renders but data appears clipped or compressed
**Cause**: Automatic scale domain doesn't fit data range
**Fix**: Explicitly set scale domains

```python
# Fix scale issues
_x_domain = [car_data['horsepower'].min(), car_data['horsepower'].max()]
_y_domain = [car_data['mpg'].min(), car_data['mpg'].max()]

_chart = alt.Chart(car_data).mark_point().encode(
    x=alt.X('horsepower:Q', scale=alt.Scale(domain=_x_domain)),
    y=alt.Y('mpg:Q', scale=alt.Scale(domain=_y_domain))
)

fixed_scale_chart = mo.ui.altair_chart(_chart)
```

### Selection Not Working

#### Missing Selection Parameters
**Symptoms**: Chart renders but no selection feedback occurs
**Cause**: Selection not added to chart or conditional encoding missing
**Fix**: Add selection and conditional encodings

```python
# Fix missing selection
_selection = alt.selection_interval()
_chart = alt.Chart(car_data).mark_point().add_params(_selection).encode(
    x='horsepower:Q',
    y='mpg:Q',
    color=alt.when(_selection).then('origin:N').otherwise(alt.value('lightgray')),
    opacity=alt.when(_selection).then(alt.value(1.0)).otherwise(alt.value(0.3))
)

working_selection_chart = mo.ui.altair_chart(_chart)
```

#### Selection Data Not Accessible
**Symptoms**: `chart.value` returns empty DataFrame despite selections
**Cause**: Chart not wrapped with `mo.ui.altair_chart()` or selections not properly configured
**Fix**: Ensure proper Marimo integration

```python
# Verify Marimo integration
if hasattr(chart, 'value'):
    _has_value_attr = True
    _value_type = type(chart.value)
else:
    _has_value_attr = False
    _value_type = None

mo.md(f"""
**Selection Debug:**
- Has .value attribute: {_has_value_attr}
- Value type: {_value_type}
- Current selection count: {len(chart.value) if _has_value_attr else 'N/A'}
""")
```

### Performance Issues

#### Slow Chart Rendering
**Symptoms**: Long delays when chart updates or loads
**Cause**: Dataset too large for client-side rendering
**Fix**: Implement data sampling or aggregation

```python
# Performance optimization
_data_size = len(data)
_max_points = 5000

if _data_size > _max_points:
    _sampled_data = data.sample(n=_max_points, random_state=42)
    _performance_note = f"Showing {_max_points} of {_data_size} points"
else:
    _sampled_data = data.copy()
    _performance_note = f"Showing all {_data_size} points"

_optimized_chart = alt.Chart(_sampled_data).mark_point().encode(
    x='value:Q', y='timestamp:T', color='category:N'
).properties(title=_performance_note)

optimized_chart = mo.ui.altair_chart(_optimized_chart)
```

#### Memory Issues with Large Datasets
**Symptoms**: Browser becomes unresponsive or crashes
**Cause**: Too much data transferred to browser
**Fix**: Server-side aggregation or progressive loading

```python
# Implement progressive data loading
_chunk_size = 1000
_total_chunks = len(large_data) // _chunk_size

chunk_selector = mo.ui.slider(
    start=0, stop=_total_chunks-1, value=0,
    label=f"Data Chunk (1-{_total_chunks})"
)

# Load data chunk based on selector
_start_idx = chunk_selector.value * _chunk_size
_end_idx = min(_start_idx + _chunk_size, len(_large_data))
_chunk_data = _large_data.iloc[_start_idx:_end_idx]

_progressive_chart = alt.Chart(_chunk_data).mark_point().encode(
    x='value:Q', y='timestamp:T', color='category:N'
).properties(
    title=f"Chunk {chunk_selector.value + 1}/{_total_chunks} (rows {_start_idx}-{_end_idx})"
)

progressive_chart = mo.ui.altair_chart(_progressive_chart)
```

### DuckDB Integration Issues

#### Connection Errors
**Symptoms**: Database connection fails or queries return errors
**Cause**: Database file missing or permission issues
**Fix**: Verify database setup and error handling

```python
import duckdb
import os

# Debug DuckDB connection
_db_path = 'analytics.db'
_db_exists = os.path.exists(_db_path)

try:
    _conn = duckdb.connect(_db_path)
    _tables = _conn.execute("SHOW TABLES").fetchall()
    _connection_success = True
    _error_message = None
except Exception as e:
    _connection_success = False
    _error_message = str(e)
    _tables = []

mo.md(f"""
**DuckDB Debug:**
- Database file exists: {_db_exists}
- Connection successful: {_connection_success}
- Available tables: {[table[0] for table in _tables]}
- Error: {_error_message}
""")
```

#### Query Performance Issues
**Symptoms**: Slow query execution affecting chart responsiveness
**Cause**: Inefficient queries or missing indexes
**Fix**: Optimize queries and add appropriate indexes

```python
# Query optimization
_optimized_query = """
    SELECT 
        DATE_TRUNC('hour', timestamp) as hour,
        category,
        AVG(value) as avg_value,
        COUNT(*) as count
    FROM transactions 
    WHERE timestamp >= ? AND timestamp <= ?
    GROUP BY DATE_TRUNC('hour', timestamp), category
    ORDER BY hour
"""

# Add query timing
import time
_start_time = time.time()
_result = _conn.execute(_optimized_query, [start_date, end_date]).fetchdf()
_query_time = time.time() - _start_time

mo.md(f"Query executed in {_query_time:.3f} seconds, returned {len(_result)} rows")
```

## Authoritative References

- [Vega-Altair Official Documentation](https://altair-viz.github.io/)
- [Vega-Altair Gallery](https://altair-viz.github.io/gallery/)
- [Vega-Lite Documentation](https://vega.github.io/vega-lite/)
- [Vega-Lite Examples](https://vega.github.io/vega-lite/examples/)
- [Marimo Documentation](https://docs.marimo.io/)
- [Marimo Interactive Elements Guide](https://docs.marimo.io/guides/interactivity/)
- [Pandas Documentation](https://pandas.pydata.org/docs/)
- [DuckDB Documentation](https://duckdb.org/docs/)