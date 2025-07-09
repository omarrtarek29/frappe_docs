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