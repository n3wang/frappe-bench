# ERPNext Development & Modification Guide

## Table of Contents
1. [Modification Objectives](#modification-objectives)
2. [Development Navigation](#development-navigation)
3. [Shared Utilities & Models](#shared-utilities--models)
4. [Inter-Module Linkage](#inter-module-linkage)
5. [Development Workflows](#development-workflows)

---

## Modification Objectives

### Primary Goals for ERPNext Simplification

#### 1. **Code Maintainability**
- **Objective**: Reduce complexity and improve code readability
- **Approach**: 
  - Consolidate redundant functionality across modules
  - Standardize naming conventions and patterns
  - Remove deprecated features and legacy code
  - Create clear separation of concerns between modules

#### 2. **Performance Optimization**
- **Objective**: Improve system responsiveness and reduce resource usage
- **Focus Areas**:
  - Database query optimization
  - Caching strategies implementation
  - Reducing unnecessary API calls
  - Optimizing client-side JavaScript bundles

#### 3. **User Experience Enhancement**
- **Objective**: Simplify workflows and reduce complexity for end users
- **Strategies**:
  - Consolidate similar forms and reduce form fields
  - Implement smart defaults and auto-population
  - Create role-based simplified interfaces
  - Reduce mandatory fields where possible

#### 4. **Customization Flexibility**
- **Objective**: Make the system easier to customize and extend
- **Implementation**:
  - Create well-defined hooks and extension points
  - Improve API consistency across modules
  - Provide clear customization guidelines
  - Enable feature toggling and configuration

---

## Development Navigation

### Quick Access Directory Structure

```bash
# Root Structure
frappe-bench/
├── apps/erpnext/erpnext/          # Main ERPNext application code
├── sites/[site-name]/             # Site-specific configurations
└── logs/                          # Application logs

# Key Development Paths
apps/erpnext/erpnext/
├── controllers/                   # Shared business logic controllers
├── accounts/                      # Financial management
├── stock/                         # Inventory management  
├── buying/                        # Purchase operations
├── selling/                       # Sales operations
├── manufacturing/                 # Production planning
├── projects/                      # Project management
├── crm/                          # Customer relationship
├── setup/                        # Configuration & setup
└── utilities/                    # Common utilities
```

### Essential Development Commands

```bash
# Navigate to ERPNext app directory
cd /home/newang/frappe-bench/apps/erpnext

# Quick access to specific modules
cd erpnext/accounts          # Financial module
cd erpnext/stock             # Inventory module  
cd erpnext/selling           # Sales module
cd erpnext/buying            # Purchase module
cd erpnext/manufacturing     # Manufacturing module
cd erpnext/controllers       # Shared controllers
cd erpnext/utilities         # Utility functions

# Development tools
bench --site [site-name] console          # Python console
bench --site [site-name] mariadb          # Database console
bench migrate                              # Run migrations
bench build                               # Build assets
bench restart                             # Restart services
```

### Key File Patterns

| Pattern | Purpose | Example |
|---------|---------|---------|
| `hooks.py` | App configuration and event hooks | `erpnext/hooks.py` |
| `__init__.py` | Module utilities and shared functions | `erpnext/stock/__init__.py` |
| `utils.py` | Module-specific utility functions | `erpnext/accounts/utils.py` |
| `doctype/*/[name].py` | Document controller classes | `erpnext/accounts/doctype/sales_invoice/sales_invoice.py` |
| `doctype/*/[name].js` | Client-side form logic | `erpnext/accounts/doctype/sales_invoice/sales_invoice.js` |
| `doctype/*/[name].json` | DocType schema definition | `erpnext/accounts/doctype/sales_invoice/sales_invoice.json` |

---

## Shared Utilities & Models

### Core Controllers Architecture (`erpnext/controllers/`)

#### 1. **Base Controllers**

**File**: `erpnext/controllers/accounts_controller.py`
- **Purpose**: Base class for all financial transactions
- **Key Features**:
  - Currency handling and conversion
  - Tax calculations
  - GL entry generation
  - Payment terms processing

```python
from erpnext.controllers.accounts_controller import AccountsController

class YourInvoiceDocType(AccountsController):
    def validate(self):
        super().validate()
        # Your custom validation
```

**File**: `erpnext/controllers/stock_controller.py`  
- **Purpose**: Base class for inventory transactions
- **Key Features**:
  - Stock ledger entry creation
  - Valuation calculations
  - Warehouse validations
  - Batch and serial number handling

**File**: `erpnext/controllers/buying_controller.py`
- **Purpose**: Base class for purchase transactions
- **Inherits**: AccountsController, StockController
- **Key Features**:
  - Supplier validation
  - Purchase price calculations
  - Purchase order workflows

**File**: `erpnext/controllers/selling_controller.py`
- **Purpose**: Base class for sales transactions  
- **Inherits**: AccountsController, StockController
- **Key Features**:
  - Customer validation
  - Pricing rule application
  - Sales order workflows

#### 2. **Specialized Controllers**

**Status Management**: `status_updater.py`
- Handles document status updates across linked documents
- Updates completion percentages in parent documents

**Tax Calculations**: `taxes_and_totals.py`
- Complex tax computation logic
- Multi-currency calculations
- Discount applications

### Shared Utilities (`erpnext/utilities/`)

**File**: `transaction_base.py`
- Base transaction functionalities
- Document linking and referencing
- Standard field calculations

**File**: `product.py`  
- Product-related utilities
- Item variant handling
- Product catalog operations

### Stock Management (`erpnext/stock/`)

**File**: `stock_ledger.py`
- Stock movement tracking
- Quantity and valuation calculations
- FIFO/Moving average implementations

**File**: `get_item_details.py`
- Item information retrieval
- Pricing and tax details
- UOM conversions

**File**: `utils.py`
- General stock utilities
- Warehouse operations
- Item categorization

### Accounts Management (`erpnext/accounts/`)

**File**: `utils.py`
- Financial calculations
- Account tree operations  
- Currency conversions
- Payment matching algorithms

**File**: `party.py`
- Customer/Supplier common operations
- Credit limit checks
- Address and contact management

---

## Inter-Module Linkage

### Module Dependencies Matrix

| Module | Depends On | Provides To |
|--------|------------|-------------|
| **Stock** | Setup, Accounts | Buying, Selling, Manufacturing |
| **Accounts** | Setup | All financial modules |
| **Buying** | Stock, Accounts | Manufacturing, Projects |
| **Selling** | Stock, Accounts, CRM | Projects |
| **Manufacturing** | Stock, Buying | - |
| **CRM** | Setup | Selling |

### Key Linking Mechanisms

#### 1. **Document References**
```python
# Sales Order → Delivery Note → Sales Invoice
sales_order = frappe.get_doc("Sales Order", "SO-001")
delivery_note = make_delivery_note(sales_order.name)
sales_invoice = make_sales_invoice(sales_order.name)
```

#### 2. **Master Data Sharing**
- **Item Master**: Shared across Stock, Buying, Selling, Manufacturing
- **Customer/Supplier**: Shared between CRM, Selling, Buying, Accounts
- **Company**: Central configuration for all modules

#### 3. **Event-Driven Updates**
```python
# In hooks.py
doc_events = {
    "Sales Order": {
        "on_submit": ["erpnext.manufacturing.planning.update_material_request"]
    }
}
```

#### 4. **Shared Workflows**
- **Procurement Process**: Buying → Stock → Accounts
- **Sales Process**: CRM → Selling → Stock → Accounts
- **Manufacturing Process**: Manufacturing → Stock → Accounts

### Data Flow Examples

#### Sales Transaction Flow
```
Lead (CRM) → Opportunity (CRM) → Quotation (Selling) → 
Sales Order (Selling) → Delivery Note (Stock) → Sales Invoice (Accounts)
```

#### Purchase Transaction Flow  
```
Material Request (Stock) → Supplier Quotation (Buying) → 
Purchase Order (Buying) → Purchase Receipt (Stock) → Purchase Invoice (Accounts)
```

#### Manufacturing Flow
```
Sales Order (Selling) → Production Plan (Manufacturing) → 
Work Order (Manufacturing) → Stock Entry (Stock) → Delivery Note (Stock)
```

---

## Development Workflows

### 1. **Setting Up Development Environment**

```bash
# Navigate to bench directory
cd /home/newang/frappe-bench

# Start development mode
bench --site [site-name] set-config developer_mode 1

# Enable debugging
bench --site [site-name] set-config auto_reload 1

# Install development dependencies
pip install frappe-bench[dev]
```

### 2. **Module Development Workflow**

#### A. Creating New Features
```bash
# 1. Navigate to appropriate module
cd apps/erpnext/erpnext/[module-name]

# 2. Create/modify DocTypes via UI or code
bench --site [site-name] console

# 3. Create controller logic
# Edit: doctype/[doctype-name]/[doctype-name].py

# 4. Add client-side logic  
# Edit: doctype/[doctype-name]/[doctype-name].js

# 5. Run migrations
bench migrate

# 6. Test changes
bench --site [site-name] run-tests
```

#### B. Modifying Existing Features
```bash
# 1. Locate the relevant files
find apps/erpnext -name "*.py" -exec grep -l "function_name" {} \;

# 2. Understand dependencies
grep -r "import.*module_name" apps/erpnext/

# 3. Make modifications
# Edit relevant .py, .js, .json files

# 4. Test thoroughly
bench --site [site-name] run-tests --module erpnext.[module]
```

### 3. **Common Customization Patterns**

#### A. Adding Custom Fields
```python
# Via Custom Field DocType or programmatically
custom_field = frappe.get_doc({
    "doctype": "Custom Field",
    "dt": "Sales Invoice",
    "fieldname": "custom_reference",
    "fieldtype": "Data", 
    "label": "Custom Reference"
})
custom_field.insert()
```

#### B. Custom Validations
```python
# In doctype controller
def validate(self):
    super().validate()
    self.custom_validation()

def custom_validation(self):
    if self.custom_field and self.custom_field < 0:
        frappe.throw("Custom field cannot be negative")
```

#### C. Custom Reports
```python
# Create in erpnext/[module]/report/[report-name]/
def execute(filters=None):
    columns = get_columns()
    data = get_data(filters)
    return columns, data
```

### 4. **Performance Optimization Workflow**

```bash
# 1. Enable query logging
bench --site [site-name] set-config enable_query_comments 1

# 2. Use profiler
bench --site [site-name] console
>>> frappe.db.sql("SHOW PROCESSLIST")

# 3. Optimize queries
# Review and optimize slow queries in logs/

# 4. Add indexes where needed
# ALTER TABLE `tabDocType` ADD INDEX `idx_field` (`field`)

# 5. Implement caching
# Use @frappe.cache() decorator for expensive operations
```

### 5. **Testing Workflow**

```bash
# Unit tests for specific module
bench --site [site-name] run-tests --module erpnext.accounts

# Integration tests
bench --site [site-name] run-tests --module erpnext.tests

# UI tests (if configured)
bench --site [site-name] run-ui-tests

# Load testing
bench --site [site-name] build
# Use external tools for load testing
```

### 6. **Debugging Workflow**

#### A. Python Debugging
```python
# Add breakpoints in code
import pdb; pdb.set_trace()

# Or use frappe debugger
frappe.log_error("Debug info", "Debug Title")

# Console debugging
bench --site [site-name] console
>>> doc = frappe.get_doc("Sales Invoice", "SI-001")
>>> doc.validate()
```

#### B. JavaScript Debugging
```javascript
// In browser console
frappe.call({
    method: "erpnext.accounts.utils.get_balance_on",
    args: {account: "Debtors - TC"},
    callback: function(r) {
        console.log(r.message);
    }
});
```

### 7. **Deployment Workflow**

```bash
# 1. Update and migrate
bench update --patch

# 2. Build assets
bench build

# 3. Clear cache
bench --site [site-name] clear-cache

# 4. Restart services  
bench restart

# 5. Run post-deployment tests
bench --site [site-name] run-tests --module erpnext.tests.test_init
```

---

## Best Practices Summary

### 1. **Code Organization**
- Follow existing module structure patterns
- Use appropriate base controllers
- Maintain clear separation between client and server logic
- Document complex business logic

### 2. **Performance**
- Minimize database queries in loops
- Use batch operations where possible
- Implement proper caching strategies
- Optimize client-side JavaScript

### 3. **Maintainability** 
- Follow ERPNext naming conventions
- Use hooks instead of core modifications
- Write comprehensive tests
- Keep customizations modular and documented

### 4. **Security**
- Validate all user inputs
- Use proper permission checks
- Sanitize data before database operations
- Follow secure coding practices

This guide provides a comprehensive foundation for understanding ERPNext's architecture and developing modifications efficiently while maintaining system integrity and performance.