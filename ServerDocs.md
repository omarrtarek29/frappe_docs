# Useful Commands Reference

## Search & File Operations

### Search in Frappe Source
```bash
# Find text in Frappe source code (excludes build files)
grep -rn "TEXT" ~/frappe-bench/apps/frappe/ \
  --exclude="*.scss" \
  --exclude="*.log" \
  --exclude="*.min.js" \
  --exclude="*.min.css" \
  --exclude="*.bundle.js" \
  --exclude="*.bundle.css" \
  --exclude-dir="dist"
```

## SSL & Security

### SSL Certificate Renewal
```bash
# Force renew SSL certificate with custom key path
sudo certbot certonly --nginx --force-renewal \
  --key-path /etc/ssl/private/new_privkey.pem \
  -d your-domain.com
```

## Git Operations

### Update Remote URL
```bash
# Change git remote URL with authentication. Using cursor, it will save authentication details, so, you don't have to confirm with key with every push.
git remote set-url origin https://username@github.com/repository
```

## Frappe Bench Commands

### Database Cleanup

```bash
# Remove tables for deleted DocTypes
bench trim-database --dry-run          # Preview what will be deleted
bench trim-database --no-backup -y     # Execute without backup
bench trim-database --format json      # JSON output

# Remove columns for deleted fields
bench trim-tables --dry-run            # Preview changes
bench trim-tables --format table       # Table format output
```

### Site Administration

```bash
# Set admin password
bench set-admin-password                # Prompts for password
bench set-admin-password newpass123     # Direct password
bench set-admin-password --logout-all-sessions  # Force logout all users

# Clear all user sessions
bench destroy-all-sessions              # Logs out all users
bench destroy-all-sessions --reason "System maintenance"  # With reason


# Site configuration
bench set-config KEY VALUE              # Set site config
bench set-config KEY VALUE --global     # Set bench config
bench set-config KEY VALUE --parse      # Parse as Python object
```

### Development Tools

```bash
# Export fixtures (default data)
bench export-fixtures                   # All apps
bench export-fixtures --app myapp       # Specific app

# Execute Python functions
bench execute your_app.utils.my_function --args "arg1,arg2"    # With arguments
bench execute your_app.utils.my_function --kwargs "key=val"    # With kwargs
bench execute your_app.utils.my_function --profile             # With profiling
```

## Quick Examples

```bash
# Clean up deleted DocTypes and fields
bench trim-database --dry-run && bench trim-tables --dry-run

# Reset admin password and force re-login
bench set-admin-password admin123 --logout-all-sessions

# Export all fixtures for backup
bench export-fixtures

# Execute custom function with parameters
bench execute "your_app.api.process_data" --args "user123" --kwargs "force=True"
```

## Supervisor Process Management

### Fix Bench Supervisor Issues
```bash
# Setup supervisor configuration
bench setup supervisor                  # Creates/updates supervisor.conf

# Reload supervisor configuration
sudo supervisorctl reread              # Read new config files
sudo supervisorctl update              # Apply configuration changes

# Restart bench processes
bench restart                          # Restart all bench processes
```

**Common Issue:** If `bench restart` fails or processes aren't running properly, run the supervisor setup sequence:

```bash
# Full supervisor reset sequence
bench setup supervisor
sudo supervisorctl reread
sudo supervisorctl update
bench restart
```

**What happens:**
- `bench setup supervisor` - Generates supervisor config for your bench
- `supervisorctl reread` - Supervisor reads the new config
- `supervisorctl update` - Applies changes and restarts affected processes
- `bench restart` - Restarts web server and background workers
