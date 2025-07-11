# Overriding Frappe Desk Pages

This guide explains how to override existing Frappe desk pages with custom HTML, CSS, and JavaScript.

## Steps to Override a Desk Page

1. Create the `www` directory in your app if it doesn't exist:
   ```bash
   mkdir -p yourapp/www
   ```

2. Create an HTML file with the same name as the page you want to override:
   ```html
   <!-- yourapp/www/desk.html -->
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>Custom Desk</title>
       
       <!-- Include Frappe's core assets -->
       {% include "templates/includes/head_tags.html" %}
       
       <!-- Your custom CSS -->
       <link rel="stylesheet" href="/assets/yourapp/css/custom_desk.css">
   </head>
   <body>
       <div class="custom-desk-container">
           <h1>Welcome to Custom Desk</h1>
           <!-- Your custom HTML content -->
       </div>
       
       <!-- Include Frappe's core scripts -->
       {% include "templates/includes/scripts.html" %}
       
       <!-- Your custom JavaScript -->
       <script src="/assets/yourapp/js/custom_desk.js"></script>
   </body>
   </html>
   ```

3. Add corresponding CSS and JavaScript files:
   ```css
   /* yourapp/public/css/custom_desk.css */
   .custom-desk-container {
       padding: 2rem;
       max-width: 1200px;
       margin: 0 auto;
   }
   ```
   
   ```javascript
   // yourapp/public/js/custom_desk.js
   frappe.ready(() => {
       // Your custom JavaScript code
       console.log('Custom desk page loaded');
   });
   ```

## Optional: Adding Python Controller

You can add a Python controller for additional server-side logic:

```python
# yourapp/www/desk.py
import frappe

def get_context(context):
    context.user = frappe.session.user
    context.custom_data = get_custom_data()
    
def get_custom_data():
    # Your custom logic here
    return {"key": "value"}
```

## Best Practices

- Keep your override files organized in appropriate directories
- Maintain Frappe's core functionality while adding custom features
- Test thoroughly across different browsers
- Consider mobile responsiveness
- Document any major changes for future maintenance

## Important Notes

1. Overriding core pages should be done carefully to avoid breaking functionality
2. Always test in a development environment first
3. Keep track of Frappe version updates that might affect your overrides
4. Consider using hooks or custom scripts instead if you only need minor modifications 