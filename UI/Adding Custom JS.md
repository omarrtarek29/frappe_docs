# Adding Custom JavaScript to Frappe Desk

This guide explains how to add custom JavaScript files to your Frappe application.

## Steps to Add Custom JS

1. Create your JavaScript file in the app's public directory:
   ```javascript
   // yourapp/public/js/custom_script.js
   frappe.ui.form.on('DocType', {
       refresh: function(frm) {
           frm.add_custom_button('My Button', () => {
               // Your custom logic here
               frappe.msgprint('Custom button clicked!');
           });
       }
   });
   ```

2. Create a bundle file in the same directory:
   ```javascript
   // yourapp/public/js/custom_script.bundle.js
   import './custom_script';
   ```

3. Include the bundle in your app's hooks.py:
   ```python
   # yourapp/hooks.py
   app_include_js = [
       "/assets/yourapp/js/custom_script.bundle.js"
   ]
   ```

## Development Workflow

1. Make changes to your JavaScript files
2. The changes will automatically reflect in development mode
3. If changes don't appear, rebuild the assets:
   ```bash
   bench build
   ```

## Best Practices

- Keep your JavaScript modular and organized
- Use meaningful file names that reflect their purpose
- Follow Frappe's JavaScript coding standards
- Test your custom JavaScript thoroughly before deployment

## Troubleshooting

If your custom JavaScript isn't working:
1. Check the browser console for errors
2. Verify the file path in hooks.py is correct
3. Clear your browser cache
4. Rebuild the assets using `bench build` 