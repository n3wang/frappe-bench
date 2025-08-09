ERPNext Item Availability Error Analysis & Solution

  🔍 Error Analysis

  The error "Item Code: 02e62bb6-7a4f-42fe-8bfe-79eece3a6491 is not 
  available under warehouse Stores - L" occurs because:

  1. The item exists in the system but has zero or negative stock in
  the "Stores - L" warehouse
  2. The item has never been stocked in this particular warehouse (no
   Bin record exists)
  3. The transaction is trying to consume/sell an item that doesn't
  have sufficient stock

  Error Source:
  - erpnext/accounts/doctype/pos_invoice/pos_invoice.py:406 - POS
  Invoice validation
  - erpnext/selling/page/point_of_sale/pos_controller.js:844 - POS
  Interface validation

  ---
  🛠️ Solutions to Fix Item Availability

  Method 1: Stock Entry (Material Receipt) - Recommended

  Step-by-Step Process:

  1. Navigate: Stock → Stock Transactions → Stock Entry
  2. Create New Stock Entry
  3. Configure:
    - Stock Entry Type: Material Receipt
    - Target Warehouse: Stores - L
    - Posting Date: Current date or desired date
  4. Add Items:
    - Item Code: 02e62bb6-7a4f-42fe-8bfe-79eece3a6491
    - Quantity: Desired stock quantity
    - Basic Rate: Cost per unit
    - Target Warehouse: Stores - L
  5. Submit the Stock Entry

  Bulk Operation for Multiple Items:

  # Execute in ERPNext Console (Advanced users)
  import frappe

  def add_items_to_warehouse(items_list, warehouse, basic_rate=100):
      stock_entry = frappe.new_doc("Stock Entry")
      stock_entry.stock_entry_type = "Material Receipt"
      stock_entry.company = "Your Company Name"  # Replace with 
  actual company

      for item_code, qty in items_list:
          stock_entry.append("items", {
              "item_code": item_code,
              "t_warehouse": warehouse,
              "qty": qty,
              "basic_rate": basic_rate,
              "uom": frappe.db.get_value("Item", item_code,
  "stock_uom")
          })

      stock_entry.save()
      stock_entry.submit()
      return stock_entry.name

  # Usage example:
  items_to_add = [
      ("02e62bb6-7a4f-42fe-8bfe-79eece3a6491", 100),
      # ("another-item-code", 50),
  ]
  stock_entry = add_items_to_warehouse(items_to_add, "Stores - L",
  100)
  print(f"Stock Entry created: {stock_entry}")

  ---
  Method 2: Stock Reconciliation - For Multiple Items

  Step-by-Step Process:

  1. Navigate: Stock → Tools → Stock Reconciliation
  2. Create New Stock Reconciliation
  3. Configure:
    - Company: Your Company
    - Purpose: Stock Reconciliation
    - Posting Date: Current date
  4. Get Items: Click "Get Items For" and select warehouse "Stores -
  L"
  5. Add/Update Items:
    - Manually add the item code:
  02e62bb6-7a4f-42fe-8bfe-79eece3a6491
    - Set Quantity: Desired stock level
    - Set Valuation Rate: Cost per unit
  6. Submit the Stock Reconciliation

  ---
  Method 3: Purchase Receipt - For New Stock

  When to Use: If items are being received from suppliers

  1. Navigate: Buying → Purchase Receipt
  2. Create Purchase Receipt
  3. Configure:
    - Supplier: Select or create supplier
    - Set Warehouse: Stores - L
  4. Add Items:
    - Item Code: 02e62bb6-7a4f-42fe-8bfe-79eece3a6491
    - Accepted Warehouse: Stores - L
    - Received Quantity: Amount received
    - Rate: Purchase rate
  5. Submit the Purchase Receipt

  ---
  🔧 Troubleshooting Steps

  Check Current Stock Status

  # ERPNext Console command
  import frappe

  item_code = "02e62bb6-7a4f-42fe-8bfe-79eece3a6491"
  warehouse = "Stores - L"

  # Check if Bin exists
  bin_exists = frappe.db.exists("Bin", {"item_code": item_code,
  "warehouse": warehouse})
  print(f"Bin exists: {bin_exists}")

  # Check current stock
  if bin_exists:
      bin_doc = frappe.get_doc("Bin", {"item_code": item_code,
  "warehouse": warehouse})
      print(f"Actual Qty: {bin_doc.actual_qty}")
      print(f"Stock Value: {bin_doc.stock_value}")
  else:
      print("No stock record found - need to create initial stock")

  Verify Item Master Data

  1. Navigate: Stock → Items and Pricing → Item
  2. Search: 02e62bb6-7a4f-42fe-8bfe-79eece3a6491
  3. Check:
    - Is Stock Item: Must be checked ✅
    - Maintain Stock: Must be enabled ✅
    - Default Warehouse: Can be set to avoid issues

  Check Warehouse Configuration

  1. Navigate: Stock → Setup → Warehouse
  2. Find: "Stores - L"
  3. Verify:
    - Is Group: Should be unchecked (must be leaf warehouse)
    - Company: Must match the company in transactions
    - Account: Should be linked to a Stock account (for perpetual
  inventory)

  ---
  ⚡ Quick Fix Commands

  Single Item Quick Add

  # Via Bench Console
  bench --site [your-site] console

  # Then execute:
import frappe
from erpnext.stock.utils import get_or_make_bin

item_code = "02e62bb6-7a4f-42fe-8bfe-79eece3a6491"
warehouse = "Stores - L"

# Create bin if doesn't exist
bin_name = get_or_make_bin(item_code, warehouse)
print(f"Bin created/exists: {bin_name}")

# Create stock entry to add quantity
stock_entry = frappe.new_doc("Stock Entry")
stock_entry.stock_entry_type = "Material Receipt"
stock_entry.company = frappe.defaults.get_user_default("Company")
stock_entry.append("items", {
    "item_code": item_code,
    "t_warehouse": warehouse,
    "qty": 100,  # Set desired quantity
    "basic_rate": 10  # Set cost per unit
})
stock_entry.save()
stock_entry.submit()
print(f"Stock Entry {stock_entry.name} created successfully")

  ---
  📊 Prevention Strategies

  1. Set Default Warehouses

  - Configure default warehouses in Item Master
  - Set company-wide default warehouses

  2. Automatic Bin Creation

  - ERPNext automatically creates Bin records when stock transactions
   occur
  - Use Stock Entry → Material Receipt for initial stock

  3. Regular Stock Reconciliation

  - Schedule periodic stock reconciliation
  - Use cycle counting for high-value items

  4. Proper Procurement Process

  - Use Purchase Orders → Purchase Receipts workflow
  - Ensure all stock receipts go through proper channels

  ---
  ✅ Verification Steps

  After implementing the solution:

  1. Check Stock Ledger: Stock → Stock Ledger → Query for the item
  and warehouse
  2. Verify Bin Record: Check if Bin record shows positive quantity
  3. Test Transaction: Try creating the original transaction that
  failed
  4. Stock Reports: Run Stock Balance report to confirm availability

  Expected Result: The error should be resolved, and the item should
  be available for transactions in the "Stores - L" warehouse.

  The Stock Entry with Material Receipt method is recommended as it
  provides proper audit trail and maintains accurate stock valuation.