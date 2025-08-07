# Marimo Notebooks Playbook

## Key Concepts

- **Reactive Execution**: Cells automatically run when their dependencies change, not in physical order
- **Pure Python Storage**: Notebooks stored as .py files, enabling Git version control and script execution
- **Global Variables**: Variables defined in cells that can be accessed by other cells (no underscore prefix)
- **Private Variables**: Variables prefixed with underscore (_) that are cell-specific and cannot be read by other cells
- **Deterministic State**: No hidden state; execution order determined by variable dependencies
- **Interactive UI Elements**: Built-in widgets for user interaction without requiring callbacks
- **Cell Dependency Graph**: Marimo tracks which cells depend on which variables to determine execution order
- **Widget Value Separation**: Widget `.value` cannot be accessed in the same cell where the widget is defined

## Common Patterns

### Default Private Variable Usage
```python
import marimo as mo
import pandas as pd
import numpy as np

# Default to private variables for intermediate work
_raw_data = pd.read_csv("data.csv")
_cleaned_data = _raw_data.dropna()
_processed_data = _cleaned_data.groupby('category').sum()

# Only expose what other cells need
final_dataset = _processed_data.reset_index()
```
**When to use**: Default approach - keep variables private unless other cells need them
**Expected output**: Only `final_dataset` is accessible globally; all intermediate work stays private

### Private Variables for Loops and Iterations
```python
import marimo as mo

# Always use private loop variables
_results = []
for _i in range(10):
    _calculation = _i ** 2
    _results.append(_calculation)

# Private comprehensions
_squared_numbers = [_x ** 2 for _x in range(10)]
_filtered_data = [_item for _item in data if _item > threshold]

# Only expose final result
processed_results = _results.copy()
```
**When to use**: All loop variables, comprehensions, and temporary iterations
**Expected output**: Loop variables don't pollute global namespace; only final results are global

### Widget Definition (Cell 1)
```python
import marimo as mo

# Define widgets - cannot access .value in same cell
user_name = mo.ui.text_input("Enter your name", placeholder="Your name here")
age_slider = mo.ui.slider(0, 100, value=25, label="Age")

# Dropdown with list options (simple labels)
category_list = mo.ui.dropdown(
    options=["Premium", "Standard", "Basic"], 
    value="Standard",  # Default to "Standard" option
    label="Membership Type"
)

# Dropdown with dictionary options (key = label, value = actual value)
priority_dict = mo.ui.dropdown(
    options={"High Priority": 1, "Medium Priority": 2, "Low Priority": 3},
    value="Medium Priority",  # IMPORTANT: value specifies the KEY (label), not the dict value
    label="Task Priority"
)

# Display widgets
mo.vstack([user_name, age_slider, category_list, priority_dict])
```

### Widget Value Usage (Cell 2)
```python
# Access widget values in separate cell
_name_value = user_name.value
_age_value = age_slider.value

# List dropdown: .value returns the selected string
_category_value = category_list.value  # Returns: "Premium", "Standard", or "Basic"

# Dictionary dropdown: .value returns the dictionary VALUE, not the key
_priority_value = priority_dict.value  # Returns: 1, 2, or 3 (the numeric values)
_priority_label = priority_dict.selected_key  # Returns: "High Priority", "Medium Priority", etc.

# Use private variables for processing
_processed_input = {
    "name": _name_value.upper() if _name_value else "Anonymous",
    "age_group": "adult" if _age_value >= 18 else "minor",
    "membership_type": _category_value,
    "priority_score": _priority_value,  # Numeric value for calculations
    "priority_display": _priority_label  # Human-readable label for display
}

# Expose only what's needed globally
user_profile = _processed_input
```
**When to use**: Widget values must be accessed in cells separate from widget definition
**Expected output**: Widget values are processed and made available to subsequent cells

## Implementation Details

### Installation and Setup
1. **Create virtual environment**:
   ```bash
   python -m venv marimo-env
   source marimo-env/bin/activate  # On Windows: marimo-env\Scripts\activate
   ```

2. **Install Marimo (recommended)**:
   ```bash
   pip install "marimo[recommended]"
   ```

3. **Verify installation**:
   ```bash
   marimo tutorial intro
   ```

**Validation**: Tutorial opens in browser showing interactive Marimo notebook

### Creating and Running Notebooks
1. **Create new notebook**:
   ```bash
   marimo edit notebook.py
   ```

2. **Run existing notebook as app**:
   ```bash
   marimo run notebook.py
   ```

3. **Convert Jupyter notebook**:
   ```bash
   marimo convert notebook.ipynb > notebook.py
   ```

**Validation**: Browser opens with Marimo interface; notebook saves as .py file

### Essential Coding Conventions

#### Variable Scoping Strategy
```python
# Cell 1: Data Loading (default to private)
_raw_sales = pd.read_csv("sales.csv")
_raw_customers = pd.read_csv("customers.csv")

# Only expose what other cells need
sales_data = _raw_sales.copy()  # If other cells need access
```

#### Processing with Private Variables
```python
# Cell 2: Data Processing (extensive private usage)
_null_count = sales_data.isnull().sum()
_duplicate_count = sales_data.duplicated().sum()

# Private loop for data cleaning
_cleaned_rows = []
for _idx, _row in sales_data.iterrows():
    _clean_row = _row.copy()
    if _row['amount'] < 0:
        _clean_row['amount'] = 0
    _cleaned_rows.append(_clean_row)

_cleaned_df = pd.DataFrame(_cleaned_rows)

# Only expose final result
clean_sales_data = _cleaned_df
```

#### UI Controls Pattern
```python
# Cell 3: UI Controls Definition
date_filter = mo.ui.date_range(label="Date Range")
amount_threshold = mo.ui.number(
    value=1000, 
    label="Minimum Amount"
)
region_select = mo.ui.multiselect(
    options=clean_sales_data['region'].unique(),
    label="Regions"
)

mo.hstack([date_filter, amount_threshold, region_select])
```

```python
# Cell 4: Apply Filters (separate cell for .value access)
_start_date, _end_date = date_filter.value
_min_amount = amount_threshold.value
_selected_regions = region_select.value

# Private filtering operations
_date_filtered = clean_sales_data[
    (clean_sales_data['date'] >= _start_date) &
    (clean_sales_data['date'] <= _end_date)
]

_amount_filtered = _date_filtered[
    _date_filtered['amount'] >= _min_amount
]

_region_filtered = _amount_filtered[
    _amount_filtered['region'].isin(_selected_regions)
] if _selected_regions else _amount_filtered

# Global result for other cells
filtered_sales = _region_filtered
```

### Dropdown Widget Patterns

#### List-Based Dropdown (Simple Options)
```python
# Cell 1: Simple string options
status_dropdown = mo.ui.dropdown(
    options=["Draft", "Published", "Archived"],
    value="Draft",  # Default selection
    label="Post Status"
)

color_dropdown = mo.ui.dropdown(
    options=["red", "green", "blue", "yellow"],
    label="Theme Color",
    searchable=True  # Enable search for long lists
)

mo.hstack([status_dropdown, color_dropdown])
```

```python
# Cell 2: Access list dropdown values
_status = status_dropdown.value  # Returns: "Draft", "Published", or "Archived"
_color = color_dropdown.value   # Returns: "red", "green", "blue", or "yellow"

# Both .value and .selected_key return the same thing for list dropdowns
_status_key = status_dropdown.selected_key  # Same as .value for lists
```

#### Dictionary-Based Dropdown (Key-Value Mapping)
```python
# Cell 3: Dictionary options - IMPORTANT: keys become labels, values are the data
plan_dropdown = mo.ui.dropdown(
    options={
        "Free Plan": {"price": 0, "features": 5},
        "Pro Plan": {"price": 29, "features": 50}, 
        "Enterprise": {"price": 99, "features": 500}
    },
    value="Pro Plan",  # CRITICAL: Use the KEY (label), not the dictionary value
    label="Subscription Plan"
)

# Numeric mapping example
difficulty_dropdown = mo.ui.dropdown(
    options={
        "Beginner": 1,
        "Intermediate": 2, 
        "Advanced": 3,
        "Expert": 4
    },
    value="Intermediate",  # Again, use KEY not number
    label="Difficulty Level"
)

mo.vstack([plan_dropdown, difficulty_dropdown])
```

```python
# Cell 4: Dictionary dropdown value access
# .value returns the dictionary VALUE (the mapped data)
_selected_plan = plan_dropdown.value        # Returns: {"price": 29, "features": 50}
_difficulty_level = difficulty_dropdown.value  # Returns: 2

# .selected_key returns the KEY (the display label)  
_plan_name = plan_dropdown.selected_key     # Returns: "Pro Plan"
_difficulty_name = difficulty_dropdown.selected_key  # Returns: "Intermediate"

# Common pattern: use both for different purposes
_subscription_info = {
    "display_name": _plan_name,           # Human-readable label
    "plan_details": _selected_plan,       # Actual data for processing
    "difficulty_display": _difficulty_name,  # For UI display
    "difficulty_score": _difficulty_level   # For calculations
}

subscription_config = _subscription_info
```

#### Dynamic Dropdown from Data
```python
# Cell 5: Create dropdown from DataFrame or data
_available_categories = clean_sales_data['category'].unique().tolist()
_category_counts = clean_sales_data['category'].value_counts().to_dict()

# Dictionary with counts for display
category_with_counts = mo.ui.dropdown(
    options={f"{cat} ({_category_counts[cat]} items)": cat for cat in _available_categories},
    label="Category (with counts)",
    searchable=True
)

# Simple list from data
region_simple = mo.ui.dropdown(
    options=clean_sales_data['region'].unique().tolist(),
    label="Region Filter"
)

mo.vstack([category_with_counts, region_simple])
```

```python
# Cell 6: Use dynamic dropdown values
_selected_category = category_with_counts.value  # Returns actual category name (no count)
_selected_region = region_simple.value           # Returns region name

# Filter data based on selections
_filtered_by_category = clean_sales_data[
    clean_sales_data['category'] == _selected_category
] if _selected_category else clean_sales_data

_final_filtered = _filtered_by_category[
    _filtered_by_category['region'] == _selected_region  
] if _selected_region else _filtered_by_category

dynamically_filtered_data = _final_filtered
```

#### Multi-Selection and Advanced Options
```python
# Cell 7: Multiple selections and advanced features
multi_tags = mo.ui.multiselect(
    options=["urgent", "bug", "feature", "documentation"],
    label="Tags (Multiple)"
)

searchable_users = mo.ui.dropdown(
    options={f"{name} ({email})": {"name": name, "email": email} 
             for name, email in [("Alice", "alice@co.com"), ("Bob", "bob@co.com")]},
    label="Assign to User",
    searchable=True,
    allow_select_none=True  # Allow deselecting
)

mo.vstack([multi_tags, searchable_users])
```

### Reactive Execution Control with mo.stop

#### Basic mo.stop Usage
```python
import marimo as mo

# Stop execution until condition is met
run_analysis = mo.ui.run_button(label="Run Analysis")

# Display button
run_analysis
```

```python
# Separate cell - stops until button is clicked
mo.stop(not run_analysis.value, mo.md("👆 Click the button to start analysis"))

# Expensive computation only runs after button click
_expensive_calculation = perform_complex_analysis(filtered_sales)
analysis_results = _expensive_calculation
```

#### Conditional Execution Control
```python
# Stop based on data validation
_data_quality_check = len(filtered_sales) > 0 and not filtered_sales.isnull().any().any()

mo.stop(
    not _data_quality_check,
    mo.md("⚠️ **Data validation failed.** Please check your filters and data quality.")
)

# Only proceeds if data passes validation
validated_data = filtered_sales
```

#### Form-based Execution Gating
```python
# Configuration form
config_form = mo.ui.form({
    "model_type": mo.ui.dropdown(["linear", "polynomial", "neural"], value="linear"),
    "train_size": mo.ui.slider(0.5, 0.9, value=0.8, step=0.1),
    "random_seed": mo.ui.number(value=42)
})

config_form
```

```python
# Stop until form is submitted
mo.stop(
    config_form.value is None,
    mo.md("📝 **Submit the configuration form** to proceed with model training")
)

# Form values only available after submission
_model_config = config_form.value
model_results = train_model(validated_data, _model_config)
```

### Altair Visualization with Interactivity

#### Reactive Chart Creation
```python
import altair as alt

# Create base chart with private variables
_base_chart = alt.Chart(filtered_sales).mark_circle(size=60).encode(
    x=alt.X('amount:Q', title='Sales Amount'),
    y=alt.Y('profit:Q', title='Profit'),
    color=alt.Color('region:N', title='Region'),
    tooltip=['product:N', 'amount:Q', 'profit:Q', 'date:T']
).properties(
    width=500,
    height=300,
    title="Sales vs Profit Analysis"
)

# Make chart interactive
sales_chart = mo.ui.altair_chart(_base_chart)
sales_chart
```

#### Chart Selection Processing
```python
# Access chart selections in separate cell
_selected_points = sales_chart.value

# Private processing of selections
_selection_summary = None
if len(_selected_points) > 0:
    _total_sales = _selected_points['amount'].sum()
    _avg_profit = _selected_points['profit'].mean()
    _regions = _selected_points['region'].unique()
    
    _selection_summary = {
        'count': len(_selected_points),
        'total_sales': _total_sales,
        'avg_profit': _avg_profit,
        'regions': list(_regions)
    }

# Global result for other cells
chart_selection_data = _selected_points
selection_summary = _selection_summary

# Display selection info
if selection_summary:
    mo.md(f"""
    **Selection Summary:**
    - Points selected: {selection_summary['count']}
    - Total sales: ${selection_summary['total_sales']:,.2f}
    - Average profit: ${selection_summary['avg_profit']:.2f}
    - Regions: {', '.join(selection_summary['regions'])}
    """)
else:
    mo.md("Select points on the chart to see summary")
```

#### Multi-Chart Dashboard
```python
# Time series chart
_time_chart = alt.Chart(filtered_sales).mark_line().encode(
    x=alt.X('date:T', title='Date'),
    y=alt.Y('sum(amount):Q', title='Total Sales'),
    color=alt.Color('region:N', title='Region')
).properties(width=500, height=200)

# Distribution chart  
_dist_chart = alt.Chart(filtered_sales).mark_bar().encode(
    x=alt.X('amount:Q', bin=True, title='Sales Amount'),
    y=alt.Y('count():Q', title='Frequency')
).properties(width=500, height=200)

# Combine charts
_combined_chart = alt.vconcat(_time_chart, _dist_chart)

dashboard_chart = mo.ui.altair_chart(_combined_chart)
dashboard_chart
```

## Validation Methods

### Reactivity Testing
- **Change upstream variable**: Modify a private variable that feeds a global variable
- **Check cell execution order**: Cells with dependencies run after their prerequisites
- **Verify private variable isolation**: Underscore variables don't appear in other cells' scope

### Widget Value Validation
```python
# Test widget separation (in separate cell from widget definition)
_widget_test = {
    'name_length': len(user_name.value) if user_name.value else 0,
    'age_valid': 0 <= age_slider.value <= 100,
    'category_selected': category_select.value is not None
}

widget_validation = _widget_test

mo.md(f"""
**Widget Validation:**
- Name length: {widget_validation['name_length']} characters
- Age valid: {widget_validation['age_valid']}  
- Category selected: {widget_validation['category_selected']}
""")
```

### Chart Interactivity Testing
```python
# Test chart selections (separate from chart definition)
_chart_test = {
    'selections_available': len(sales_chart.value) > 0,
    'selection_columns': list(sales_chart.value.columns) if len(sales_chart.value) > 0 else [],
    'data_types_preserved': True
}

mo.md(f"Chart has {len(sales_chart.value)} selected points with columns: {_chart_test['selection_columns']}")
```

## Anti-patterns to Avoid

### ❌ Using Global Variables for Temporary Work
```python
# DON'T DO THIS - pollutes global namespace
temp_sum = sum(numbers)
temp_mean = temp_sum / len(numbers)  
intermediate_calc = temp_mean * 1.1
final_result = intermediate_calc
```

### ✅ Use Private Variables for All Intermediate Work
```python
# DO THIS - keep intermediate work private
_temp_sum = sum(numbers)
_temp_mean = _temp_sum / len(numbers)
_intermediate_calc = _temp_mean * 1.1

# Only expose final result
final_result = _intermediate_calc
```

### ❌ Using Global Loop Variables
```python
# DON'T DO THIS - loop variables become global
results = []
for i in range(10):  # 'i' becomes global
    calculation = i ** 2  # 'calculation' becomes global
    results.append(calculation)
```

### ✅ Always Use Private Loop Variables
```python
# DO THIS - keep loop variables private
_results = []
for _i in range(10):  # Private loop variable
    _calculation = _i ** 2  # Private intermediate
    _results.append(_calculation)

# Only expose final result
results = _results.copy()
```

### ❌ Accessing Widget Values in Same Cell
```python
# DON'T DO THIS - won't work
slider = mo.ui.slider(0, 100, value=50)
calculation = slider.value * 2  # ERROR: value not available yet
```

### ✅ Separate Widget Definition and Value Usage  
```python
# Cell 1: Define widget
slider = mo.ui.slider(0, 100, value=50)
slider

# Cell 2: Use widget value
_slider_value = slider.value
calculation = _slider_value * 2
```

### ❌ Mutating Global Variables
```python
# Cell 1
data = [1, 2, 3]

# Cell 2 - DON'T DO THIS
data.append(4)  # Mutation not tracked by reactivity
```

### ✅ Create New Variables Instead
```python
# Cell 1  
_original_data = [1, 2, 3]
data = _original_data.copy()

# Cell 2 - DO THIS
_extended_data = data + [4]
updated_data = _extended_data
```

### ❌ Dropdown Dictionary Value Parameter Confusion
```python
# DON'T DO THIS - using dictionary value instead of key
options_dict = {"High": 1, "Medium": 2, "Low": 3}
priority = mo.ui.dropdown(
    options=options_dict,
    value=2,  # WRONG: 2 is a dictionary VALUE, not a key
    label="Priority"
)
```

### ✅ Use Dictionary Keys (Labels) for Value Parameter
```python
# DO THIS - use dictionary key (the label) for value parameter
options_dict = {"High": 1, "Medium": 2, "Low": 3}
priority = mo.ui.dropdown(
    options=options_dict,
    value="Medium",  # CORRECT: "Medium" is a key/label in the dictionary
    label="Priority"
)

# Access the actual mapped value in separate cell
_priority_score = priority.value  # Returns: 2 (the mapped value)
_priority_label = priority.selected_key  # Returns: "Medium" (the key/label)
```

### ❌ Misunderstanding Dictionary Dropdown Value Access
```python
# Dropdown with dictionary options
category_dropdown = mo.ui.dropdown(
    options={"Category A": "cat_a", "Category B": "cat_b"},
    value="Category A"
)

# DON'T DO THIS - expecting .value to return the key
if category_dropdown.value == "Category A":  # WRONG: .value returns "cat_a", not "Category A"
    process_category_a()
```

### ✅ Use Correct Properties for Dictionary Dropdowns  
```python
# DO THIS - understand what each property returns
category_dropdown = mo.ui.dropdown(
    options={"Category A": "cat_a", "Category B": "cat_b"},
    value="Category A"
)

# In separate cell:
_selected_value = category_dropdown.value        # Returns: "cat_a" (dictionary value)
_selected_label = category_dropdown.selected_key  # Returns: "Category A" (dictionary key)

# Use the appropriate property for your logic
if _selected_label == "Category A":    # Compare with selected_key for labels
    process_category_a()

if _selected_value == "cat_a":         # Compare with value for mapped data
    handle_category_a_data()
```

## Debugging and Troubleshooting

### Variable Scope Issues

#### Private Variables Not Accessible
**Symptoms**: `NameError` when trying to access underscore variable from another cell
**Cause**: Attempting to use private variable globally
**Fix**: Remove underscore or create global version

```python
# Cell 1
_temp_calc = expensive_operation()
public_result = _temp_calc  # Create global version

# Cell 2
final_output = public_result + 10  # Use global version
```

#### Widget Value Access Errors
**Symptoms**: Widget appears but accessing `.value` gives unexpected results
**Cause**: Trying to access `.value` in same cell as widget definition
**Fix**: Move `.value` access to separate cell

```python
# Cell 1: Widget definition only
user_input = mo.ui.text_input("Enter text")
user_input

# Cell 2: Value access only
_input_value = user_input.value
processed_input = _input_value.upper() if _input_value else ""
```

### Execution Control Issues

#### Cells Running When They Shouldn't
**Symptoms**: Expensive computations run automatically
**Cause**: No execution gating
**Fix**: Use `mo.stop()` with conditions

```python
# Add execution gate
run_expensive = mo.ui.run_button(label="Run Expensive Calculation")

mo.stop(
    not run_expensive.value,
    mo.md("Click button to run expensive calculation")
)

_expensive_result = very_expensive_function()
result = _expensive_result
```

#### Circular Dependencies
**Symptoms**: Cells fail to run; error about circular dependency
**Cause**: Variable A depends on B, and B depends on A
**Fix**: Use private variables to break the cycle

```python
# ✅ Break circular dependency with private variables
_intermediate = process_x(input_data)
result_a = transform_a(_intermediate)
result_b = transform_b(_intermediate)  # Both depend on intermediate, not each other
```

### Chart Interaction Issues

#### Chart Selections Not Updating
**Symptoms**: Altair chart renders but selections don't update Python variables
**Cause**: Not wrapping with `mo.ui.altair_chart()`
**Fix**: Wrap chart properly

```python
# ❌ Static chart
_chart = alt.Chart(data).mark_circle()

# ✅ Interactive chart  
_chart = alt.Chart(data).mark_circle()
interactive_chart = mo.ui.altair_chart(_chart)
```

#### No Selection Data Available
**Symptoms**: `chart.value` is empty even after selections
**Cause**: Chart definition doesn't support selections
**Fix**: Add selection capability to chart

```python
# Add brush selection to enable data selection
_chart = alt.Chart(data).mark_circle().encode(
    x='x:Q', 
    y='y:Q'
).add_selection(
    alt.selection_interval(bind='scales')
)

interactive_chart = mo.ui.altair_chart(_chart)
```

## mo.stop Function Reference

### Function Signature
```python
mo.stop(predicate: bool, output: Optional[object] = None) -> None
```

### Parameters
- **predicate**: Boolean condition that triggers stopping when `True`
- **output**: Optional output to display when execution stops (default: `None`)

### Basic Usage Patterns
```python
# Stop until button is clicked
mo.stop(not button.value, mo.md("Click the button to continue"))

# Stop if form is not submitted
mo.stop(form.value is None, mo.md("Submit the form to proceed"))

# Stop on validation failure
mo.stop(not is_valid, mo.md("Fix validation errors before continuing"))
```

### Advanced Control Patterns
```python
# Multiple conditions
_data_loaded = data is not None
_user_authenticated = user.is_authenticated
_prerequisites_met = _data_loaded and _user_authenticated

mo.stop(
    not _prerequisites_met,
    mo.md("Ensure data is loaded and user is authenticated")
)

# Conditional expensive operations
_should_run_analysis = analysis_button.value and len(selected_data) > 0

mo.stop(
    not _should_run_analysis,
    mo.md("Select data and click 'Run Analysis' to proceed")
)
```

### Error Prevention
```python
# Prevent division by zero
_divisor = user_number.value
mo.stop(_divisor == 0, mo.md("⚠️ Divisor cannot be zero"))

_result = numerator / _divisor
calculation_result = _result
```

## Authoritative References

- [Marimo Official Documentation](https://docs.marimo.io/)
- [Marimo Reactivity Guide](https://docs.marimo.io/guides/reactivity/)
- [Marimo Best Practices](https://docs.marimo.io/guides/best_practices/)
- [Marimo Interactive Elements Guide](https://docs.marimo.io/guides/interactivity/)
- [Marimo Control Flow Documentation](https://docs.marimo.io/api/control_flow/)
- [Marimo Installation Guide](https://docs.marimo.io/getting_started/installation.html)
- [Altair Visualization Grammar](https://altair-viz.github.io/)