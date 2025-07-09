# Frappe Fixtures Configuration Guide

## Development Tools

### Export Fixtures (Default Data)
```bash
# Export all fixtures from all apps
bench export-fixtures

# Export fixtures from specific app
bench export-fixtures --app myapp

# Export specific DocType fixtures
bench export-fixtures --app myapp --doctype "Custom DocType"
```

## Fixture Configuration

Fixtures allow you to export and import default data that should be available across all instances of your app. Configure fixtures in your app's `hooks.py`:

### Basic Fixture Setup

```python
# In hooks.py
fixtures = [
    "Custom DocType",           # Export all records
    "Item Group",              # Export all Item Groups
    "UOM",                     # Export all Units of Measure
]
```

### Advanced Fixture Configuration with Filters

```python
# In hooks.py
fixtures = [
    # Simple: Export all records of a DocType
    "Custom DocType",
    "Print Format",
    
    # Complex: Export with specific filters
    {
        "dt": "Role",                    # DocType name
        "filters": [
            ["name", "in", ("Sales User", "Purchase User", "Stock User")]
        ]
    }
]
``` 