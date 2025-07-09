# Frappe Form Functions Reference

## Document Navigation
```javascript
// Switch to different document of same doctype
frm.switch_doc("DOC-2024-001");

// Example: Navigate to next document in sequence
frappe.ui.form.on("Sales Order", {
    refresh: function(frm) {
        if (!frm.is_new()) {
            frm.add_custom_button("Next Order", () => {
                frappe.db.get_list("Sales Order", {
                    filters: { creation: [">", frm.doc.creation] },
                    limit: 1,
                    order_by: "creation asc"
                }).then(docs => {
                    if (docs.length > 0) {
                        frm.switch_doc(docs[0].name);
                    }
                });
            });
        }
    }
});
```

## Save Controls
```javascript
// Enable/disable save functionality
frm.enable_save();                    // Allow saving
frm.disable_save();                   // Prevent saving  
frm.disable_save(true);               // Prevent saving but allow dirty state
frm.disable_form();                   // Make entire form read-only
```

## Document Operations
```javascript
// Reload document from database
frm.reload_doc();

// Refresh specific field
frm.refresh_field("field_name");

// Example: Auto-refresh after async operation
frappe.ui.form.on("Item", {
    update_stock: function(frm) {
        frappe.call({
            method: "update_inventory",
            args: { item_code: frm.doc.item_code },
            callback: function() {
                frm.reload_doc(); // Refresh entire form
                frm.refresh_field("stock_qty"); // Or just specific field
            }
        });
    }
});
```

## Permission Checks
```javascript
// Check user permissions
frm.has_perm("read");     // Can read document
frm.has_perm("write");    // Can edit document  
frm.has_perm("create");   // Can create new document
frm.has_perm("delete");   // Can delete document
frm.has_perm("submit");   // Can submit document

// Example: Conditional button based on permissions
frappe.ui.form.on("Purchase Order", {
    refresh: function(frm) {
        if (frm.has_perm("submit") && frm.doc.docstatus === 0) {
            frm.add_custom_button("Quick Submit", () => {
                frm.submit();
            });
        }
    }
});
```

## Document State
```javascript
// Check document state
frm.is_dirty();          // Has unsaved changes
frm.is_new();            // Is new document (not saved yet)
frm.dirty();             // Mark document as dirty
frm.get_docinfo();       // Get document metadata

// Example: Warn before navigation if dirty
frappe.ui.form.on("Customer", {
    before_route: function(frm) {
        if (frm.is_dirty()) {
            frappe.msgprint("You have unsaved changes!");
            return false; // Prevent navigation
        }
    }
});
```

## Custom Buttons
```javascript
// Add custom buttons
frm.add_custom_button("Button Label", function() {
    // Button action
}, "Button Group");

// Remove buttons
frm.clear_custom_buttons();           // Remove all custom buttons
frm.remove_custom_button("Label");    // Remove specific button
frm.remove_custom_button("Label", "Group"); // Remove from group

// Example: Dynamic buttons based on status
frappe.ui.form.on("Task", {
    refresh: function(frm) {
        frm.clear_custom_buttons();
        
        if (frm.doc.status === "Open") {
            frm.add_custom_button("Start Work", () => {
                frm.set_value("status", "Working");
                frm.save();
            }, "Actions");
        }
        
        if (frm.doc.status === "Working") {
            frm.add_custom_button("Complete", () => {
                frm.set_value("status", "Completed");
                frm.save();
            }, "Actions");
        }
    }
});
```

## Visual Indicators

```javascript
frm.set_indicator_formatter("target_field", (item) => (item.condition ? "success_color" : "warning_color"));
```

**Purpose**: Display visual status indicators in child table rows

**Pattern**: `field_name → condition_check → color_result`

**Logic**: Evaluates row data and returns appropriate status color

## Dynamic ChildTable Field Filtering

```javascript
frm.fields_dict.child_table.grid.get_field("dropdown_field").get_query = function (doc, cdt, cdn) {
    var current_row = locals[cdt][cdn];
    return {
        filters: {
            criteria_field: current_row.reference_value,
        },
    };
};
```

**Purpose**: Filter dropdown options based on row context

**Pattern**: `child_table → target_field → filter_criteria → filtered_options`

**Logic**: Uses current row values to dynamically filter available options

## User Tracking

```javascript
frm.get_involved_users()
```

**Purpose**: Retrieve document interaction history

**Pattern**: `document → user_interactions → user_list`

**Logic**: Returns collection of users with document involvement

## Navigation Control

```javascript
frm.scroll_to_field("target_field");
```

**Purpose**: Programmatic viewport navigation

**Pattern**: `field_identifier → scroll_action → field_visibility`

**Logic**: Moves viewport to bring specified field into view

## Keyboard Shortcuts

```javascript
frappe.ui.keys.add_shortcut({
    shortcut: "key_combination",
    action: () => custom_function(),
    page: frm.page,
    description: __("Action Description"),
    ignore_inputs: boolean,
    condition: () => validation_check(),
});
```

**Purpose**: Register custom keyboard interactions

**Pattern**: `key_combo → condition_check → action_execution`

**Logic**: Binds keyboard events to custom functions with optional conditions

## Complete Form Example

```javascript
frappe.ui.form.on("Sales Invoice", {
    refresh: function(frm) {
        // Check permissions and add buttons
        if (frm.has_perm("write") && !frm.is_new()) {
            frm.add_custom_button("Send Email", () => {
                send_invoice_email(frm);
            }, "Actions");
            
            frm.add_custom_button("Print PDF", () => {
                if (frm.is_dirty()) {
                    frappe.toast({
                        message: "Please save changes before printing",
                        indicator: "yellow"
                    });
                    return;
                }
                frappe.set_route("print", frm.doctype, frm.doc.name);
            }, "Actions");
        }
        
        // Auto-refresh customer balance
        if (frm.doc.customer) {
            frm.refresh_field("outstanding_amount");
        }
    },
    
    customer: function(frm) {
        // Mark form dirty when customer changes
        frm.dirty();
    }
});

function send_invoice_email(frm) {
    frappe.call({
        method: "send_invoice_by_email",
        args: {
            invoice_name: frm.doc.name,
            recipient: frm.doc.customer_email
        },
        callback: function(response) {
            if (response.message) {
                frappe.toast({
                    message: "Invoice emailed successfully",
                    indicator: "green"
                });
                frm.reload_doc(); // Refresh to show email status
            }
        }
    });
}
``` 