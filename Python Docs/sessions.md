# Session Hooks - Custom Login Redirects

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

## Hook Registration
Add to `hooks.py`:
```python
# Trigger custom redirect logic on every login
on_session_creation = "your_app.overrides.session_hooks.on_session_creation"
```

## Example Use Cases:
- Customer portal users → `/portal`
- Delivery drivers → `/driver-dashboard`  
- Managers → `/reports-page`
- Regular users → `/app` (default) 