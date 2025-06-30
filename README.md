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
