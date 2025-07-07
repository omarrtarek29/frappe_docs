# Frappe Form Functions Reference

### Document Navigation
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

### Save Controls
```javascript
// Enable/disable save functionality
frm.enable_save();                    // Allow saving
frm.disable_save();                   // Prevent saving  
frm.disable_save(true);               // Prevent saving but allow dirty state
frm.disable_form();                   // Make entire form read-only
```

### Document Operations
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

### Permission Checks
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

### Document State
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

### Custom Buttons
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

### Visual Indicators

```javascript
frm.set_indicator_formatter("target_field", (item) => (item.condition ? "success_color" : "warning_color"));
```

**Purpose**: Display visual status indicators in child table rows

**Pattern**: `field_name → condition_check → color_result`

**Logic**: Evaluates row data and returns appropriate status color

---

### Dynamic ChildTable Field Filtering

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

---

### User Tracking

```javascript
frm.get_involved_users()
```

**Purpose**: Retrieve document interaction history

**Pattern**: `document → user_interactions → user_list`

**Logic**: Returns collection of users with document involvement

---

### Navigation Control

```javascript
frm.scroll_to_field("target_field");
```

**Purpose**: Programmatic viewport navigation

**Pattern**: `field_identifier → scroll_action → field_visibility`

**Logic**: Moves viewport to bring specified field into view

---

### Keyboard Shortcuts

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

## Global Functions

### Toast Messages
```javascript
// Show toast notifications
frappe.toast({
    message: "Operation completed successfully",
    indicator: "green"        // green, red, yellow, blue
});

frappe.toast({
    message: __("Document has unsaved changes.<br>Consider saving before printing."),
    indicator: "yellow",
    title: "Warning"          // Optional title
});

// Example: Success feedback after API call
frappe.call({
    method: "send_email",
    args: { recipient: frm.doc.email },
    callback: function(response) {
        if (response.message.success) {
            frappe.toast({
                message: "Email sent successfully",
                indicator: "green"
            });
        }
    }
});
```

### Navigation
```javascript
// Navigate to different routes
frappe.set_route("List", "Customer");              // Go to list view
frappe.set_route("Form", "Customer", "CUST-001");  // Go to specific form
frappe.set_route("print", "Sales Order", "SO-001"); // Go to print view
frappe.set_route("query-report", "Financial Statements"); // Go to report

// Example: Navigate after form submission
frappe.ui.form.on("Lead", {
    after_save: function(frm) {
        if (frm.doc.status === "Converted") {
            frappe.set_route("Form", "Customer", frm.doc.customer);
        }
    }
});
```

## Complete Example

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

# Frappe ListView Settings - Quick Reference

## Basic Structure

```javascript
frappe.listview_settings['DocType Name'] = {
    // Settings go here
};
```

## Essential Properties

### Control Visibility
```javascript
{
    can_create: false,              // Hide create button
    hide_name_column: true,         // Hide ID column
    add_fields: ['field1', 'field2'], // Fetch additional fields
}
```

### Event Handlers
```javascript
{
    onload: function(listview) {
        // Runs when list loads - most customizations go here
    },
    
    before_render: function() {
        // Runs before each render
    },
    
    primary_action: function() {
        // Custom create button action
        frappe.new_doc('DocType Name', { default_field: 'value' });
    }
}
```

## Field Indicators

```javascript
{
    get_indicator: function(doc) {
        // Status colors: red, green, blue, orange, yellow, purple, darkgrey
        if (doc.status === "Active") {
            return [__('Active'), 'green', 'status,=,Active'];
        }
        if (doc.status === "Pending") {
            return [__('Pending'), 'orange', 'status,=,Pending'];
        }
        if (doc.status === "Cancelled") {
            return [__('Cancelled'), 'red', 'status,=,Cancelled'];
        }
        // Format: [display_text, color, filter_condition]
    }
}
```

## Custom Field Display

```javascript
{
    formatters: {
        amount: function(value, df, doc) {
            return `<strong>$${value}</strong>`;
        },
        
        priority: function(value, df, doc) {
            const colors = { High: 'red', Medium: 'orange', Low: 'green' };
            return `<span style="color: ${colors[value]}">${value}</span>`;
        },
        
        status_date: function(value, df, doc) {
            if (!value) return '';
            return frappe.datetime.str_to_user(value);
        }
    }
}
```

## Custom Buttons

### Single Row Button
```javascript
{
    button: {
        show: function(doc) {
            return doc.status === "Draft";
        },
        get_label: function(doc) {
            return __('Process');
        },
        get_description: function(doc) {
            return __('Process this record');
        },
        action: function(doc) {
            frappe.call({
                method: 'app_name.api.process_doc',
                args: { name: doc.name },
                callback: function() {
                    cur_list.refresh();
                }
            });
        }
    }
}
```

### Page Buttons
```javascript
{
    onload: function(listview) {
        // Add button to page toolbar
        listview.page.add_inner_button(__('Custom Action'), function() {
            frappe.msgprint(__('Custom action executed'));
        });
        
        // Add bulk action to actions menu
        listview.page.add_actions_menu_item(__('Bulk Process'), function() {
            let selected = listview.get_checked_items(true);
            if (selected.length === 0) {
                frappe.msgprint(__('Please select items'));
                return;
            }
            
            frappe.call({
                method: 'app_name.api.bulk_process',
                args: { names: selected }
            });
        });
    }
}
```

## Filters & Search

```javascript
{
    onload: function(listview) {
        // Add default filters
        listview.filter_area.add(listview.doctype, 'status', '!=', 'Cancelled');
        listview.filter_area.add(listview.doctype, 'creation', '>', '2024-01-01');
        
        // Conditional filters
        if (frappe.user.has_role('Manager')) {
            listview.filter_area.add(listview.doctype, 'assigned_to', '=', frappe.session.user);
        }
        
        // Hide sidebar
        if (listview.list_sidebar) {
            listview.list_sidebar.parent.hide();
        }
    }
}
```

## Quick Tips

```javascript
// Access current list view
cur_list

// Refresh list
cur_list.refresh()

// Get selected items
listview.get_checked_items()      // Full objects
listview.get_checked_items(true)  // Just names

// Add filters programmatically
listview.filter_area.add(doctype, field, operator, value)

// Remove filters  
listview.filter_area.remove(field_name)

// Available colors for indicators
// red, green, blue, orange, yellow, purple, darkgrey, lightblue
```

## Common Use Cases

### Hide Create Button for Specific Users
```javascript
{
    onload: function(listview) {
        if (!frappe.user.has_role('Manager')) {
            listview.can_create = false;
            listview.page.clear_primary_action();
        }
    }
}
```

### Auto-refresh List
```javascript
{
    onload: function(listview) {
        setInterval(() => {
            if (frappe.get_route_str().includes(listview.doctype)) {
                listview.refresh();
            }
        }, 30000); // Refresh every 30 seconds
    }
}
```

### Custom New Document
```javascript
{
    primary_action: function() {
        let dialog = new frappe.ui.Dialog({
            title: __('New Record'),
            fields: [
                {fieldtype: 'Data', fieldname: 'title', label: __('Title'), reqd: 1}
            ],
            primary_action: function() {
                let values = dialog.get_values();
                frappe.new_doc('Your DocType', values);
                dialog.hide();
            }
        });
        dialog.show();
    }
}
```


# Frappe Breadcrumbs - Custom Navigation

## Purpose
Fix Frappe's breadcrumb navigation to remember the last workspace and maintain proper navigation flow.

## Method 1: Per DocType (List/Form JS Files)

### List View JS
```javascript
frappe.listview_settings["Your DocType"] = {
  refresh: function (listview) {
    const previous_route = frappe.get_prev_route();
    let stored_workspace = localStorage.getItem("current_workspace") || "";
    
    if (previous_route && previous_route[0] === "Workspaces") {
      stored_workspace = previous_route[1];
      localStorage.setItem("current_workspace", stored_workspace);
    }
    
    if (stored_workspace) {
      frappe.breadcrumbs.clear();
      frappe.breadcrumbs.all[frappe.get_route_str()] = {
        workspace: stored_workspace,
        doctype: listview.doctype,
        type: "List",
      };
      frappe.breadcrumbs.update();
    }
  },
};
```

### Form View JS
```javascript
frappe.ui.form.on("Your DocType", {
  refresh: function(frm) {
    const stored_workspace = localStorage.getItem("current_workspace");
    
    if (stored_workspace) {
      frappe.breadcrumbs.clear();
      frappe.breadcrumbs.all[frappe.get_route_str()] = {
        workspace: stored_workspace,
        doctype: frm.doctype,
        type: "Form",
      };
      frappe.breadcrumbs.update();
    }
  },
});
```

## Method 2: Global Override (App Hooks)

### Global Breadcrumb Override
```javascript

(function() {
    const original_method = frappe.breadcrumbs.update;
    
    frappe.breadcrumbs.update = function() {
        const active_route = frappe.get_route();
        const stored_workspace = localStorage.getItem("current_workspace");
        
        if (stored_workspace) {
            // Handle Form view
            if (active_route[0] === "Form") {
                const doc_type = active_route[1];
                const doc_name = active_route[2];
                
                this.clear();
                
                // Workspace > DocType List > Form
                this.append_breadcrumb_element(
                    `/app/${frappe.router.slug(stored_workspace)}`,
                    __(stored_workspace)
                );
                
                this.append_breadcrumb_element(
                    `/app/${frappe.router.slug(doc_type)}`,
                    __(doc_type)
                );
                
                // Handle new vs existing documents
                let document_title;
                if (doc_name.startsWith("new-" + doc_type.toLowerCase().replace(/ /g, "-"))) {
                    document_title = __("New {0}", [__(doc_type)]);
                } else {
                    document_title = __(doc_name);
                }
                
                this.append_breadcrumb_element(
                    `/app/${frappe.router.slug(doc_type)}/${encodeURIComponent(doc_name)}`,
                    document_title
                );
                
                // Make last breadcrumb copyable
                let final_crumb = this.$breadcrumbs.find("li").last();
                final_crumb.addClass("disabled").css("cursor", "copy");
                final_crumb.click((event) => {
                    event.stopImmediatePropagation();
                    frappe.utils.copy_to_clipboard(final_crumb.text());
                });
                
                this.toggle(true);
                return;
            } 
            // Handle List view  
            else if (active_route[0] === "List") {
                const doc_type = active_route[1];
                
                this.clear();
                
                // Workspace > DocType List
                this.append_breadcrumb_element(
                    `/app/${frappe.router.slug(stored_workspace)}`,
                    __(stored_workspace)
                );
                
                this.append_breadcrumb_element(
                    `/app/${frappe.router.slug(doc_type)}`,
                    __(doc_type)
                );
                
                this.toggle(true);
                return;
            }
            else if (active_route[0] === "Workspaces") {
                localStorage.setItem("current_workspace", active_route[1]);
            }
        }
        
        // Fall-back
        original_method.call(this);
    };
})();
```

### Add to App Hooks
```python
# In hooks.py
app_include_js = [
    "assets/js/navigation_breadcrumbs.js"
]
```

## What This Does

**Problem:** Frappe's default breadcrumbs lose workspace context when navigating between forms and lists.

**Solution:** 
- Tracks the last visited workspace in localStorage
- Rebuilds breadcrumbs: `Workspace > DocType List > Form`
- Works for both individual DocTypes and globally

**Navigation Flow:**
```
Workspace → List View → Form View
    ↑         ↑          ↑
  Tracked   Shows WS   Shows WS > List
```

**Bonus Features:**
- Click last breadcrumb to copy document name
- Handles "New Document" vs existing documents
- Maintains proper navigation hierarchy
