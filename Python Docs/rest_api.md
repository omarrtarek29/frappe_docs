# 🚀 Frappe REST API Quick Reference

*Master REST APIs in Frappe with this lightning-fast guide*

## 🏗️ Basic Setup

### File Structure
```
your_app/
├── your_app/
│   ├── api/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── product.py
│   └── hooks.py
```

### Basic API Template
```python
# your_app/api/user.py
import frappe
from frappe import _

@frappe.whitelist()
def get_user_profile(user_id):
    """GET /api/method/your_app.api.user.get_user_profile"""
    
    # Validation
    if not user_id:
        frappe.response["http_status_code"] = 400
        return {"error": "user_id is required"}
    
    # Logic
    user = frappe.get_doc("User", user_id)
    
    # Response
    frappe.response["http_status_code"] = 200
    return {
        "user": {
            "name": user.name,
            "email": user.email,
            "full_name": user.full_name
        }
    }
```

---

## 🔧 HTTP Methods

### GET - Retrieve Data
```python
@frappe.whitelist(methods=["GET"])
def get_products(limit=10, offset=0, category=None):
    """GET /api/method/your_app.api.product.get_products"""
    
    filters = {}
    if category:
        filters["category"] = category
    
    products = frappe.get_all(
        "Product",
        filters=filters,
        fields=["name", "title", "price", "category"],
        limit=int(limit),
        start=int(offset)
    )
    
    return {
        "products": products,
        "total": len(products),
        "limit": limit,
        "offset": offset
    }
```

### POST - Create Data
```python
@frappe.whitelist(methods=["POST"])
def create_product():
    """POST /api/method/your_app.api.product.create_product"""
    
    data = frappe.local.form_dict
    
    # Validation
    required_fields = ["title", "price", "category"]
    for field in required_fields:
        if not data.get(field):
            frappe.response["http_status_code"] = 400
            return {"error": f"{field} is required"}
    
    try:
        # Create document
        product = frappe.get_doc({
            "doctype": "Product",
            "title": data.get("title"),
            "price": float(data.get("price")),
            "category": data.get("category"),
            "description": data.get("description", "")
        })
        
        product.insert()
        
        frappe.response["http_status_code"] = 201
        return {
            "message": "Product created successfully",
            "product_id": product.name,
            "product": product.as_dict()
        }
        
    except Exception as e:
        frappe.response["http_status_code"] = 500
        return {"error": str(e)}
```

### PUT - Update Data
```python
@frappe.whitelist(methods=["PUT"])
def update_product(product_id):
    """PUT /api/method/your_app.api.product.update_product"""
    
    if not product_id:
        frappe.response["http_status_code"] = 400
        return {"error": "product_id is required"}
    
    data = frappe.local.form_dict
    
    try:
        product = frappe.get_doc("Product", product_id)
        
        # Update fields
        if data.get("title"):
            product.title = data.get("title")
        if data.get("price"):
            product.price = float(data.get("price"))
        if data.get("category"):
            product.category = data.get("category")
        
        product.save()
        
        frappe.response["http_status_code"] = 200
        return {
            "message": "Product updated successfully",
            "product": product.as_dict()
        }
        
    except frappe.DoesNotExistError:
        frappe.response["http_status_code"] = 404
        return {"error": "Product not found"}
    except Exception as e:
        frappe.response["http_status_code"] = 500
        return {"error": str(e)}
```

### DELETE - Remove Data
```python
@frappe.whitelist(methods=["DELETE"])
def delete_product(product_id):
    """DELETE /api/method/your_app.api.product.delete_product"""
    
    if not product_id:
        frappe.response["http_status_code"] = 400
        return {"error": "product_id is required"}
    
    try:
        frappe.delete_doc("Product", product_id)
        
        frappe.response["http_status_code"] = 204
        return {"message": "Product deleted successfully"}
        
    except frappe.DoesNotExistError:
        frappe.response["http_status_code"] = 404
        return {"error": "Product not found"}
    except Exception as e:
        frappe.response["http_status_code"] = 500
        return {"error": str(e)}
```

---

## 🔐 Authentication & Permissions

### Basic Authentication Check
```python
@frappe.whitelist()
def protected_endpoint():
    """Requires authentication"""
    
    if frappe.session.user == "Guest":
        frappe.response["http_status_code"] = 401
        return {"error": "Authentication required"}
    
    return {"message": "Welcome, authenticated user!"}
```

### Role-Based Access
```python
@frappe.whitelist()
def admin_only_endpoint():
    """Only for administrators"""
    
    if not frappe.only_for("System Manager"):
        frappe.response["http_status_code"] = 403
        return {"error": "Admin access required"}
    
    return {"message": "Admin-only content"}
```

### Custom Permission Check
```python
@frappe.whitelist()
def check_user_permission(doctype, name):
    """Check if user has permission for specific document"""
    
    if not frappe.has_permission(doctype, "read", name):
        frappe.response["http_status_code"] = 403
        return {"error": "Insufficient permissions"}
    
    doc = frappe.get_doc(doctype, name)
    return {"document": doc.as_dict()}
```

---

## 📦 Request & Response Patterns

### Handle JSON Payload
```python
@frappe.whitelist(methods=["POST"])
def handle_json_payload():
    """Handle JSON request body"""
    
    try:
        # Get JSON data
        data = frappe.local.form_dict
        
        # Alternative: Parse JSON from request
        # import json
        # data = json.loads(frappe.local.request.data)
        
        # Process data
        result = process_user_data(data)
        
        return {"success": True, "data": result}
        
    except json.JSONDecodeError:
        frappe.response["http_status_code"] = 400
        return {"error": "Invalid JSON payload"}
```

### Pagination Pattern
```python
@frappe.whitelist()
def get_paginated_data(page=1, per_page=20, search=None):
    """Standard pagination pattern"""
    
    page = int(page)
    per_page = int(per_page)
    start = (page - 1) * per_page
    
    filters = {}
    if search:
        filters["title"] = ["like", f"%{search}%"]
    
    # Get data
    data = frappe.get_all(
        "Product",
        filters=filters,
        fields=["name", "title", "price"],
        limit=per_page,
        start=start,
        order_by="creation desc"
    )
    
    # Get total count
    total = frappe.db.count("Product", filters)
    
    return {
        "data": data,
        "pagination": {
            "page": page,
            "per_page": per_page,
            "total": total,
            "pages": (total + per_page - 1) // per_page
        }
    }
```

### File Upload Handling
```python
@frappe.whitelist(methods=["POST"])
def upload_file():
    """Handle file uploads"""
    
    if not frappe.local.uploaded_file:
        frappe.response["http_status_code"] = 400
        return {"error": "No file uploaded"}
    
    try:
        file_doc = frappe.get_doc({
            "doctype": "File",
            "file_name": frappe.local.uploaded_filename,
            "content": frappe.local.uploaded_file,
            "is_private": 0
        })
        
        file_doc.insert()
        
        frappe.response["http_status_code"] = 201
        return {
            "message": "File uploaded successfully",
            "file_url": file_doc.file_url,
            "file_name": file_doc.file_name
        }
        
    except Exception as e:
        frappe.response["http_status_code"] = 500
        return {"error": str(e)}
```

---

## ⚠️ Error Handling

### Comprehensive Error Handler
```python
def handle_api_error(func):
    """Decorator for consistent error handling"""
    
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        
        except frappe.DoesNotExistError:
            frappe.response["http_status_code"] = 404
            return {"error": "Resource not found"}
        
        except frappe.PermissionError:
            frappe.response["http_status_code"] = 403
            return {"error": "Insufficient permissions"}
        
        except frappe.ValidationError as e:
            frappe.response["http_status_code"] = 422
            return {"error": str(e)}
        
        except Exception as e:
            frappe.log_error(frappe.get_traceback())
            frappe.response["http_status_code"] = 500
            return {"error": "Internal server error"}
    
    return wrapper

# Usage
@frappe.whitelist()
@handle_api_error
def my_api_endpoint():
    # Your API logic here
    pass
```

### Validation Pattern
```python
def validate_required_fields(data, required_fields):
    """Validate required fields"""
    
    missing_fields = []
    for field in required_fields:
        if not data.get(field):
            missing_fields.append(field)
    
    if missing_fields:
        frappe.response["http_status_code"] = 400
        return {
            "error": "Missing required fields",
            "missing_fields": missing_fields
        }
    
    return None

# Usage
@frappe.whitelist(methods=["POST"])
def create_user():
    data = frappe.local.form_dict
    
    # Validate
    error = validate_required_fields(data, ["email", "first_name"])
    if error:
        return error
    
    # Process...
```

---

## 🚀 Advanced Patterns

### Bulk Operations
```python
@frappe.whitelist(methods=["POST"])
def bulk_create_products():
    """Create multiple products in one request"""
    
    data = frappe.local.form_dict
    products_data = data.get("products", [])
    
    if not products_data:
        frappe.response["http_status_code"] = 400
        return {"error": "No products provided"}
    
    created_products = []
    errors = []
    
    for idx, product_data in enumerate(products_data):
        try:
            product = frappe.get_doc({
                "doctype": "Product",
                "title": product_data.get("title"),
                "price": product_data.get("price"),
                "category": product_data.get("category")
            })
            
            product.insert()
            created_products.append(product.name)
            
        except Exception as e:
            errors.append({
                "index": idx,
                "error": str(e),
                "data": product_data
            })
    
    return {
        "created": len(created_products),
        "errors": len(errors),
        "created_products": created_products,
        "error_details": errors
    }
```

### Search & Filter API
```python
@frappe.whitelist()
def search_products(q=None, category=None, min_price=None, max_price=None):
    """Advanced search with multiple filters"""
    
    filters = []
    
    # Text search
    if q:
        filters.append(["title", "like", f"%{q}%"])
    
    # Category filter
    if category:
        filters.append(["category", "=", category])
    
    # Price range
    if min_price:
        filters.append(["price", ">=", float(min_price)])
    if max_price:
        filters.append(["price", "<=", float(max_price)])
    
    products = frappe.get_all(
        "Product",
        filters=filters,
        fields=["name", "title", "price", "category"],
        order_by="title asc"
    )
    
    return {
        "products": products,
        "total": len(products),
        "filters_applied": {
            "search": q,
            "category": category,
            "price_range": [min_price, max_price]
        }
    }
```

### Rate Limiting
```python
import time
from frappe.utils import now_datetime

# Simple in-memory rate limiter (use Redis in production)
rate_limit_cache = {}

def rate_limit(max_requests=100, window_seconds=3600):
    """Rate limiting decorator"""
    
    def decorator(func):
        def wrapper(*args, **kwargs):
            user = frappe.session.user
            current_time = time.time()
            
            if user not in rate_limit_cache:
                rate_limit_cache[user] = []
            
            # Clean old requests
            rate_limit_cache[user] = [
                req_time for req_time in rate_limit_cache[user]
                if current_time - req_time < window_seconds
            ]
            
            # Check limit
            if len(rate_limit_cache[user]) >= max_requests:
                frappe.response["http_status_code"] = 429
                return {"error": "Rate limit exceeded"}
            
            # Add current request
            rate_limit_cache[user].append(current_time)
            
            return func(*args, **kwargs)
        
        return wrapper
    return decorator

# Usage
@frappe.whitelist()
@rate_limit(max_requests=10, window_seconds=60)
def limited_endpoint():
    return {"message": "Success"}
```

---

## ✅ Best Practices

### 1. API Response Structure
```python
# ✅ Good - Consistent response structure
def good_api_response():
    return {
        "success": True,
        "data": {...},
        "message": "Operation completed successfully",
        "metadata": {
            "timestamp": frappe.utils.now(),
            "version": "1.0"
        }
    }

# ❌ Bad - Inconsistent structure
def bad_api_response():
    return {...}  # Raw data without structure
```

### 2. Error Response Format
```python
# ✅ Good - Structured error response
def handle_error():
    frappe.response["http_status_code"] = 400
    return {
        "success": False,
        "error": {
            "code": "VALIDATION_ERROR",
            "message": "Invalid input data",
            "details": {...}
        }
    }
```

### 3. API Documentation
```python
@frappe.whitelist()
def well_documented_api(user_id, include_details=False):
    """
    Get user information by ID
    
    Args:
        user_id (str): The user ID to fetch
        include_details (bool): Whether to include detailed information
    
    Returns:
        dict: User information with the following structure:
            {
                "user": {
                    "id": str,
                    "name": str,
                    "email": str,
                    "details": dict (optional)
                }
            }
    
    Raises:
        404: User not found
        403: Insufficient permissions
    """
    pass
```

### 4. Version Your APIs
```python
# your_app/api/v1/user.py
@frappe.whitelist()
def get_user():
    """Version 1 of user API"""
    pass

# your_app/api/v2/user.py
@frappe.whitelist()
def get_user():
    """Version 2 with enhanced features"""
    pass
```

---

## 🎯 Quick Reference

### Status Codes
- `200` - OK (successful GET, PUT)
- `201` - Created (successful POST)
- `204` - No Content (successful DELETE)
- `400` - Bad Request (validation errors)
- `401` - Unauthorized (authentication required)
- `403` - Forbidden (insufficient permissions)
- `404` - Not Found (resource doesn't exist)
- `422` - Unprocessable Entity (validation errors)
- `429` - Too Many Requests (rate limiting)
- `500` - Internal Server Error

### Common Patterns
```python
# Get form data
data = frappe.local.form_dict

# Set status code
frappe.response["http_status_code"] = 201

# Check permissions
if not frappe.has_permission("DocType", "read"):
    return {"error": "No permission"}

# Handle exceptions
try:
    # logic
except frappe.DoesNotExistError:
    # handle not found
```

