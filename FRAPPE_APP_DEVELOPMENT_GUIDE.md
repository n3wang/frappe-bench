# Frappe Framework - App Development Guide

## Overview

This comprehensive guide analyzes the Frappe Framework architecture and identifies reusable components, utilities, and patterns that are essential for developing robust Frappe applications.

## Framework Architecture

### Directory Structure

The Frappe Framework follows a well-organized structure:

```
frappe-bench/
├── apps/                 # All Frappe applications
│   ├── frappe/          # Core Frappe Framework
│   ├── erpnext/         # Reference ERP application
│   └── custom_apps/     # Your custom applications
├── sites/               # Site-specific configurations and data
├── config/              # Bench configuration files
└── logs/                # Application logs
```

### Core Framework Components

#### 1. Document Model System (`frappe/model/`)

**Key Files:**
- `document.py` - Base Document class for all DocTypes
- `base_document.py` - Controller system for custom business logic
- `naming.py` - Document naming strategies

**Usage Pattern:**
```python
# In your custom DocType controller
import frappe
from frappe.model.document import Document

class YourDocType(Document):
    def validate(self):
        # Custom validation logic
        pass
    
    def before_save(self):
        # Logic before saving
        pass
    
    def on_submit(self):
        # Logic after submission
        pass
```

#### 2. Utility Functions (`frappe/utils/`)

**Essential Utilities:**

**Data Manipulation:**
- `flt()` - Safe float conversion
- `cint()` - Safe integer conversion
- `cstr()` - Safe string conversion
- `get_fullname()` - User display name formatting
- `validate_email_address()` - Email validation

**Date/Time:**
- `nowdate()`, `now()` - Current timestamps
- `getdate()` - Date parsing
- `add_days()`, `add_months()` - Date arithmetic
- `get_datetime()` - DateTime operations

**File Operations:**
- `get_files_path()` - File system paths
- `get_bench_path()` - Bench directory path
- `execute_in_shell()` - Safe shell command execution

#### 3. Database Abstraction (`frappe/database/`)

**Key Features:**
- ORM-like query building with `frappe.qb` (Query Builder)
- Safe parameterized queries
- Transaction management
- Database schema management

**Example Usage:**
```python
# Using Query Builder
from frappe import qb

# Select query
customers = qb.DocType("Customer")
query = qb.from_(customers).select(customers.name, customers.customer_name).where(customers.disabled == 0)
results = query.run(as_dict=True)

# Using frappe.db shortcuts
frappe.db.get_value("Customer", "CUST-001", "customer_name")
frappe.db.get_list("Customer", filters={"disabled": 0}, fields=["name", "customer_name"])
```

## Reusable Patterns and Components

### 1. Controller Architecture Pattern

**Base Controllers in ERPNext (`erpnext/controllers/`):**

- `accounts_controller.py` - Financial transaction base
- `buying_controller.py` - Purchase transaction base  
- `selling_controller.py` - Sales transaction base
- `stock_controller.py` - Inventory transaction base
- `status_updater.py` - Status management base

**Implementation Example:**
```python
from erpnext.controllers.selling_controller import SellingController

class YourSalesDocType(SellingController):
    def validate(self):
        super().validate()
        # Your custom validation
        self.calculate_custom_fields()
    
    def calculate_custom_fields(self):
        # Custom calculations building on base controller
        pass
```

### 2. Hooks System

**Core Hook Categories (from `frappe/hooks.py`):**

**Document Events:**
```python
doc_events = {
    "*": {
        "after_insert": ["your_app.module.function"],
        "on_update": ["your_app.module.function"],
        "before_save": ["your_app.module.function"],
        "on_submit": ["your_app.module.function"],
    },
    "Specific DocType": {
        "validate": ["your_app.module.validate_function"]
    }
}
```

**Scheduled Events:**
```python
scheduler_events = {
    "daily": ["your_app.tasks.daily_cleanup"],
    "hourly": ["your_app.tasks.hourly_sync"],
    "weekly": ["your_app.tasks.weekly_report"],
    "monthly": ["your_app.tasks.monthly_summary"],
    "cron": {
        "0/15 * * * *": ["your_app.tasks.every_15_minutes"]
    }
}
```

**Web Routes:**
```python
website_route_rules = [
    {"from_route": "/custom/<path:name>", "to_route": "custom_page"},
]
```

### 3. Permissions Framework

**Permission Query Conditions:**
```python
permission_query_conditions = {
    "Your DocType": "your_app.permissions.get_permission_query_conditions"
}

has_permission = {
    "Your DocType": "your_app.permissions.has_permission"
}
```

**Example Implementation:**
```python
def get_permission_query_conditions(user):
    if "System Manager" in frappe.get_roles(user):
        return ""
    return f"(`tabYour DocType`.owner = '{user}' or `tabYour DocType`.assigned_to = '{user}')"
```

### 4. Client-Side Controllers

**JavaScript Patterns:**
```javascript
frappe.ui.form.on('Your DocType', {
    refresh: function(frm) {
        // Form refresh logic
        if (frm.doc.status === "Draft") {
            frm.add_custom_button("Custom Action", function() {
                // Custom button logic
            });
        }
    },
    
    validate: function(frm) {
        // Client-side validation
        if (!frm.doc.required_field) {
            frappe.throw("Required field is mandatory");
        }
    },
    
    custom_field: function(frm) {
        // Field change triggers
        frm.set_value("calculated_field", calculate_value(frm.doc));
    }
});
```

### 5. Utility Patterns

**Common Helper Functions:**

**Validation Utilities:**
```python
from frappe.utils import validate_email_address, validate_phone_number, validate_url

# Email validation
email = validate_email_address("user@example.com", throw=True)

# Phone validation  
is_valid_phone = validate_phone_number("+1234567890")

# URL validation
is_valid_url = validate_url("https://example.com", throw=True)
```

**Data Processing:**
```python
from frappe.utils import get_fullname, get_formatted_email, sanitize_html

# User display formatting
display_name = get_fullname("user@example.com")
formatted_email = get_formatted_email("user@example.com")

# Content sanitization
clean_html = sanitize_html(user_input, linkify=True)
```

### 6. Error Handling Patterns

**Structured Exception Handling:**
```python
import frappe

try:
    # Business logic
    result = process_data()
except frappe.ValidationError as e:
    # Validation-specific handling
    frappe.log_error(f"Validation failed: {e}")
    frappe.throw(str(e))
except Exception as e:
    # Generic error handling
    frappe.log_error(f"Unexpected error: {e}")
    frappe.throw("An unexpected error occurred")
```

### 7. API Development Patterns

**Whitelisted Methods:**
```python
import frappe

@frappe.whitelist()
def get_customer_details(customer):
    """Public API method"""
    return frappe.get_doc("Customer", customer).as_dict()

@frappe.whitelist(allow_guest=True)
def public_api_method():
    """Method accessible to guests"""
    return {"status": "success"}
```

## Development Best Practices

### 1. App Structure
```
your_app/
├── your_app/
│   ├── __init__.py
│   ├── hooks.py              # App configuration and hooks
│   ├── modules.txt           # Module list
│   ├── patches.txt           # Data migration patches
│   ├── config/               # App configuration
│   ├── public/               # Static assets (JS, CSS)
│   ├── templates/            # Jinja templates
│   ├── www/                  # Web pages
│   └── your_module/          # Business logic modules
│       ├── __init__.py
│       ├── doctype/          # DocType definitions
│       └── utils.py          # Utility functions
├── requirements.txt          # Python dependencies
├── setup.py                  # Package configuration
└── README.md
```

### 2. DocType Development

**Controller Best Practices:**
- Always call `super()` methods in overrides
- Use proper validation in `validate()` method
- Handle permissions in controller methods
- Implement proper error handling
- Use database transactions for complex operations

### 3. Performance Optimization

**Database Queries:**
```python
# Efficient: Batch operations
frappe.db.sql("""
    UPDATE `tabItem` 
    SET disabled = 1 
    WHERE item_group = %s
""", (item_group,))

# Efficient: Use indexes
frappe.db.get_list("Sales Order", 
    filters={"status": "Pending", "customer": customer_name},
    fields=["name", "grand_total"],
    order_by="creation desc",
    limit=10
)
```

**Caching:**
```python
# Use built-in caching
@frappe.cache()
def get_expensive_calculation(param):
    # Expensive operation
    return result
```

### 4. Testing Patterns

**Unit Testing:**
```python
import frappe
import unittest

class TestYourDocType(unittest.TestCase):
    def setUp(self):
        # Test data setup
        self.test_doc = frappe.get_doc({
            "doctype": "Your DocType",
            "field1": "test_value"
        }).insert()
    
    def test_validation(self):
        # Test validation logic
        with self.assertRaises(frappe.ValidationError):
            invalid_doc = frappe.get_doc({
                "doctype": "Your DocType",
                "field1": ""  # Invalid value
            })
            invalid_doc.save()
    
    def tearDown(self):
        # Cleanup
        self.test_doc.delete()
```

## Security Best Practices

### 1. Input Validation
- Always validate and sanitize user inputs
- Use parameterized queries to prevent SQL injection
- Sanitize HTML content with `sanitize_html()`

### 2. Permission Checks
- Implement proper permission query conditions
- Use `has_permission()` methods for access control
- Validate user roles before sensitive operations

### 3. Data Protection
```python
# Safe data access
if frappe.has_permission("DocType", "read", doc.name):
    return doc.as_dict()
else:
    frappe.throw("Insufficient permissions")
```

## Advanced Patterns

### 1. Event-Driven Architecture
```python
# Custom event emission
frappe.publish_realtime("custom_event", {
    "message": "Processing complete",
    "doctype": "Your DocType",
    "docname": doc.name
})

# Event handling in JavaScript
frappe.realtime.on("custom_event", function(data) {
    frappe.show_alert(data.message);
});
```

### 2. Background Jobs
```python
# Queue background job
frappe.enqueue("your_app.tasks.long_running_task", 
               timeout=300,
               queue="long",
               job_name="process_bulk_data",
               **kwargs)
```

### 3. Custom Print Formats
```python
# In hooks.py
additional_print_settings = "your_app.printing.get_print_settings"

# Implementation
def get_print_settings():
    return {
        "print_style": "your_custom_style.css",
        "print_settings": {
            "compact_print": 1
        }
    }
```

## Migration and Deployment

### 1. Data Patches
```python
# In patches.txt
your_app.patches.v1_0.migrate_customer_data

# Patch implementation
import frappe

def execute():
    # Data migration logic
    customers = frappe.get_all("Customer", fields=["name"])
    for customer in customers:
        # Update logic
        frappe.db.set_value("Customer", customer.name, "new_field", "default_value")
```

### 2. Schema Updates
```python
def execute():
    # Add new fields to existing DocTypes
    from frappe.custom.doctype.custom_field.custom_field import create_custom_field
    
    create_custom_field("Customer", {
        "label": "Customer Rating",
        "fieldname": "customer_rating",
        "fieldtype": "Select",
        "options": "Excellent\nGood\nAverage\nPoor"
    })
```

## Conclusion

The Frappe Framework provides a robust foundation for building enterprise applications with:

- **Comprehensive MVC Architecture** - Clear separation of concerns
- **Rich Utility Library** - Extensive helper functions for common tasks  
- **Flexible Hook System** - Event-driven customization capabilities
- **Security Framework** - Built-in permission and validation systems
- **Database Abstraction** - ORM-like query building and schema management
- **Client-Server Integration** - Seamless JavaScript-Python communication

By leveraging these patterns and components, developers can build maintainable, scalable, and secure applications efficiently. The framework's modular design allows for easy customization while maintaining consistency across different modules and applications.

For detailed API documentation, refer to the official Frappe Framework documentation at https://frappeframework.com/docs.