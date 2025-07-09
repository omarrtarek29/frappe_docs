# Frappe Navigation & UI Controls

## Navigation Functions

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

## Toast Messages

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
        else {
            if (active_route[0] === "Workspaces") {
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