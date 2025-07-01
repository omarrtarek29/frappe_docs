# Frappe Logging - Quick Reference

## Setup

```python
import frappe
from frappe.utils.logger import set_log_level

# Set GLOBAL log level - affects ALL loggers in your Frappe site
set_log_level("DEBUG")    # Shows everything (DEBUG, INFO, WARNING, ERROR, CRITICAL)
set_log_level("INFO")     # Shows INFO, WARNING, ERROR, CRITICAL (hides DEBUG)
set_log_level("WARNING")  # Shows WARNING, ERROR, CRITICAL (hides DEBUG, INFO)
set_log_level("ERROR")    # Shows ERROR, CRITICAL only
set_log_level("CRITICAL") # Shows CRITICAL only

# Create logger (inherits the global level set above)
logger = frappe.logger(name, allow_site=True, file_count=50)
```

## Parameters

### `frappe.logger(name, allow_site, file_count)`
- **`name`** (str): Log file name (creates `{name}.log`)
- **`allow_site`** (bool): Enable site-specific logging 
- **`file_count`** (int): Number of log files to keep (rotation)

### `set_log_level(level)`
- **`level`** (str): `"DEBUG"`, `"INFO"`, `"WARNING"`, `"ERROR"`, `"CRITICAL"`

## Log Levels (Severity Order)

```python
logger = frappe.logger("my_app", allow_site=True, file_count=10)

# 1. DEBUG - Detailed diagnostic info (lowest level)
logger.debug("Processing item: ABC123, quantity: 5")
logger.debug("Database query executed in 0.05s")

# 2. INFO - General operational messages
logger.info("Order SO-001 created successfully")
logger.info("Email sent to customer@example.com")

# 3. WARNING - Something unexpected but not breaking
logger.warning("Low stock alert: Item ABC123 has only 2 remaining")
logger.warning("API rate limit approaching: 90/100 calls used")

# 4. ERROR - Serious problems that need attention
logger.error("Failed to send email: SMTP server unreachable")
logger.error("Payment gateway returned error code 402")

# 5. CRITICAL - System-breaking issues (highest level)
logger.critical("Database connection lost")
logger.critical("Out of memory - system shutting down")

# EXCEPTION - Error + full stack trace (use in except blocks)
try:
    result = 1/0
except Exception as e:
    logger.exception("Division by zero error occurred")  # Shows full traceback
```

## When Each Level Shows Up

If you set `set_log_level("WARNING")`:
- ✅ WARNING, ERROR, CRITICAL will be logged
- ❌ DEBUG, INFO will be ignored

If you set `set_log_level("DEBUG")`:
- ✅ All levels will be logged

## File Location
```
sites/{site_name}/logs/{name}.log
```

## Common Pattern

```python
def my_function():
    logger = frappe.logger("module_name", allow_site=True, file_count=30)
    
    try:
        logger.info("Starting process")
        # Your code here
        logger.info("Process completed")
    except Exception as e:
        logger.error(f"Process failed: {e}")
        logger.exception("Full error details")
```

## Session Hooks - Custom Login Redirects

**Purpose:** Redirect different user roles to specific pages after login instead of default `/app`

```python
import frappe

def on_session_creation(login_manager):
    """Redirect users to role-specific pages after login"""
    user_roles = frappe.get_roles(frappe.session.user)
    
    if "Custom Role" in user_roles:
        frappe.local.response["home_page"] = "/custom-page"
    elif "Another Role" in user_roles:
        frappe.local.response["home_page"] = "/another-page"  
    else:
        frappe.local.response["home_page"] = "/app"  # Default Frappe desk
```

### Hook Registration
Add to `hooks.py`:
```python
# Trigger custom redirect logic on every login
on_session_creation = "your_app.overrides.session_hooks.on_session_creation"
```

**Example Use Cases:**
- Customer portal users → `/portal`
- Delivery drivers → `/driver-dashboard`  
- Managers → `/reports-page`
- Regular users → `/app` (default)



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

# Fixture Configuration

Fixtures allow you to export and import default data that should be available across all instances of your app. Configure fixtures in your app's `hooks.py`:

## Basic Fixture Setup

```python
# In hooks.py
fixtures = [
    "Custom DocType",           # Export all records
    "Item Group",              # Export all Item Groups
    "UOM",                     # Export all Units of Measure
]
```

## Advanced Fixture Configuration with Filters

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

